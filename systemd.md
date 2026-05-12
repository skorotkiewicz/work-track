## Systemd

Runtime state lives in tmpfs. To limit data loss to ~5 minutes on sudden reboot, add a systemd user timer:

```ini
# ~/.config/systemd/user/worktrack-save.timer
[Unit]
Description=Periodic work-track save

[Timer]
OnBootSec=5min
OnUnitActiveSec=5min

[Install]
WantedBy=timers.target
```

```ini
# ~/.config/systemd/user/worktrack-save.service
[Unit]
Description=Bank work-track session

[Service]
Type=oneshot
ExecStart=%h/.local/bin/work-track save
```

Enable:

```bash
systemctl --user daemon-reload
systemctl --user enable --now worktrack-save.timer
```
