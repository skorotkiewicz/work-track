# work-track

A minimal screen-on time tracker for Wayland.

Tracks how long your screen is active each day. Hooks into `swayidle` to detect screen on/off and suspend/resume events, persists daily totals to JSON, and correctly handles midnight boundaries.

---

## State

- **Runtime:** `$XDG_RUNTIME_DIR/worktrack/` (tmpfs, no disk writes during tracking)
- **Persist:** `$XDG_DATA_HOME/worktrack/YYYY-MM-DD.json`

## Usage

```bash
work-track on       # screen on  (swayidle resume / after-resume)
work-track off      # screen off (swayidle timeout / before-sleep)
work-track status   # print today's total
work-track save     # bank current session, persist to JSON
work-track log      # show last 7 days
mini-track stats N  # last N days (default: 1)
```

## Integration

Add to your compositor startup config (e.g., Niri):

```rust
spawn-sh-at-startup "swayidle -w \\
    timeout 300 'swaylock -f' \\
    timeout 600 'work-track off; niri msg action power-off-monitors' \\
        resume 'work-track on' \\
    before-sleep 'work-track off' \\
    after-resume 'work-track on' &"
```

- `resume` → monitor wake
- `after-resume` → system resume

## Why RAM state?

Runtime state lives in `$XDG_RUNTIME_DIR` (tmpfs/RAM) instead of writing straight to the JSON for two reasons:

1. **Zero disk wear:** If `work-track status` is used in a Waybar that updates every 5 seconds, writing to JSON would wear out your SSD. tmpfs allows instant, zero-wear reads/writes.
2. **Crash safety:** If the system loses power while the screen is active, the in-memory state is lost. This prevents writing an incorrect “always-on” session to disk, which would otherwise corrupt historical accuracy. At most, a small amount of unrecorded time is lost, while existing daily JSON data remains valid. A systemd timer reduces this gap to roughly ~5 minutes.

   See [systemd](systemd.md) for implementation details.

---

## Waybar

```json
{
  "custom/worktrack": {
    "exec": "work-track status",
    "interval": 60,
    "format": "⏱ {}"
  }
}
```

---

## JSON Output

```json
{
  "date": "2026-05-12",
  "seconds": 13335
}
```
