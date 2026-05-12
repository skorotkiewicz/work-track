# work-track

A minimal screen-on time tracker for Wayland.

Tracks how long your monitors are powered on each day. Hooks into `swayidle` to detect screen on/off and suspend/resume events, persists daily totals to JSON, and gracefully handles midnight crossovers and system crashes.

## State

- **Runtime:** `$XDG_RUNTIME_DIR/worktrack/` (tmpfs, zero disk wear)
- **Persist:** `$XDG_DATA_HOME/worktrack/YYYY-MM-DD.json`

## Usage

```
work-track on       screen on  (swayidle resume / after-resume)
work-track off      screen off (swayidle timeout / before-sleep)
work-track status   print today's total
work-track save     bank current session, persist to JSON
work-track log      show last 7 days
```

## Integration

Add to your compositor startup config (e.g., Niri):

```bash
spawn-sh-at-startup "swayidle -w \
    timeout 300 'swaylock -f -i /usr/share/backgrounds/archlinux/snow.jpg' \
    timeout 600 'work-track off; niri msg action power-off-monitors' \
        resume 'work-track on' \
    before-sleep 'work-track off; swaylock -f -i /usr/share/backgrounds/archlinux/snow.jpg' \
    after-resume 'work-track on' &"
```

Note: `resume` handles waking from monitor idle, while `after-resume` handles waking from system suspend. Both are required for accurate tracking.

## Why RAM state?

Runtime state lives in `$XDG_RUNTIME_DIR` (tmpfs/RAM) instead of writing straight to the JSON for two reasons:

1. **Zero disk wear:** If `work-track status` is used in a Waybar that updates every 5 seconds, writing to JSON would wear out your SSD. tmpfs allows instant, zero-wear reads/writes. We only hit the disk when the state actually changes (monitor off, save).
2. **Crash safety:** If the system loses power while the screen is on, the RAM state safely vanishes. Writing "currently on" to disk could leave the JSON in a broken "forever on" state on crash. This way, only a few minutes of unbanked time are lost, and the historical JSON remains valid. (The [systemd](systemd.md) timer mitigates this loss to ~5 minutes).

## Waybar

```json
"custom/worktrack": {
    "exec": "work-track status",
    "interval": 60,
    "format": "⏱ {}"
}
```

## JSON Output

`~/.local/share/worktrack/2025-05-12.json`:

```json
{
  "date": "2025-05-12",
  "total_seconds": 13335,
  "total_hours": 3.70,
  "human": "3h 42m 15s"
}
```
