# systemd / systemctl Cheatsheet

## Service control
| Command | What it does |
|---|---|
| `systemctl status foo` | Status of `foo` service |
| `systemctl start foo` | Start now |
| `systemctl stop foo` | Stop now |
| `systemctl restart foo` | Stop then start |
| `systemctl reload foo` | Reload config (no restart, if supported) |
| `systemctl enable foo` | Start on boot |
| `systemctl disable foo` | Don't start on boot |
| `systemctl enable --now foo` | Enable AND start now |
| `systemctl mask foo` | Forbid starting at all |
| `systemctl unmask foo` | Allow again |

## Inspection
| Command | What it does |
|---|---|
| `systemctl list-units` | All loaded units |
| `systemctl list-units --type=service` | Just services |
| `systemctl list-units --state=running` | Just running ones |
| `systemctl list-units --state=failed` | What's broken |
| `systemctl list-unit-files` | All unit files (including disabled) |
| `systemctl cat foo` | Show foo's unit file(s) |
| `systemctl show foo` | All properties of foo |
| `systemctl get-default` | Current default target |
| `systemctl is-active foo` | Just yes/no |
| `systemctl is-enabled foo` | Boot status |

## Editing units
| Command | What it does |
|---|---|
| `systemctl edit foo` | Create a drop-in override |
| `systemctl edit --full foo` | Edit the whole unit (copy to /etc/) |
| `systemctl daemon-reload` | After editing units (always remember!) |

## Targets (≈ runlevels)
| Command | What it does |
|---|---|
| `systemctl list-units --type=target` | Loaded targets |
| `systemctl get-default` | Default target |
| `systemctl set-default multi-user.target` | Change default |
| `systemctl isolate rescue.target` | Switch now (careful!) |

## Timers
| Command | What it does |
|---|---|
| `systemctl list-timers` | All timers, when they fire next |
| `systemctl status foo.timer` | Status |

## journalctl
| Command | What it does |
|---|---|
| `journalctl` | All logs |
| `journalctl -u foo` | Just foo's logs |
| `journalctl -u foo -f` | Follow live |
| `journalctl -b` | Current boot |
| `journalctl -b -1` | Previous boot |
| `journalctl --since "1 hour ago"` | Time-filtered |
| `journalctl --since today --until "1 hour ago"` | Range |
| `journalctl -p err` | Errors and worse |
| `journalctl -k` | Kernel only (like dmesg) |
| `journalctl --disk-usage` | Size on disk |
| `journalctl --vacuum-time=7d` | Delete older than 7 days |

## Priorities (numbers and names)
- 0 emerg
- 1 alert
- 2 crit
- 3 err
- 4 warning
- 5 notice
- 6 info
- 7 debug

## Minimal service unit

```ini
[Unit]
Description=My thing
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/my-script.sh
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

## Minimal timer unit

```ini
[Unit]
Description=Run my-thing periodically

[Timer]
OnBootSec=5min
OnUnitActiveSec=1h
Persistent=true

[Install]
WantedBy=timers.target
```

(Pair with a `.service` file of the same name.)

---

[← Back to course home](../README.md)
