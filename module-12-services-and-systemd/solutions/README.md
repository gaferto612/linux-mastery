# Module 12 — Solutions

## Exercise 1 — Your first service

`/usr/local/bin/heartbeat.sh` and `/etc/systemd/system/heartbeat.service` as in the exercise. The activation ritual:

```bash
sudo systemctl daemon-reload          # tell systemd to re-read unit files
sudo systemctl start heartbeat        # run it now
sudo systemctl status heartbeat       # is it healthy?
sudo journalctl -u heartbeat -f       # follow its output
sudo systemctl enable heartbeat       # symlink into WantedBy=multi-user.target.wants
```

What `enable` actually does:

```bash
ls -l /etc/systemd/system/multi-user.target.wants/
# heartbeat.service -> /etc/systemd/system/heartbeat.service
```

That symlink is why the service comes up next boot. `disable` removes the symlink.

The Type matters:

```
Type=simple      → ExecStart is the long-running process (default)
Type=forking     → process forks and parent exits; systemd tracks the child
Type=oneshot     → runs to completion, then "exited" is the steady state
Type=notify      → process tells systemd "I'm ready" via sd_notify
```

---

## Exercise 2 — Stop, restart, disable

```bash
sudo systemctl stop heartbeat         # send SIGTERM, escalate to SIGKILL after timeout
sudo systemctl restart heartbeat
sudo systemctl disable heartbeat      # won't start at boot
sudo systemctl is-enabled heartbeat   # disabled
sudo systemctl is-active heartbeat    # inactive
```

`stop` (one-shot) vs `disable` (permanent boot-time):

- `stop` + still enabled → starts again on next boot.
- `disable` + still running → keeps running until next reboot / stop.
- `disable --now` does both.

---

## Exercise 3 — Timers

Two units that pair up:

```ini
# /etc/systemd/system/say-hello.service
[Unit]
Description=Says hello

[Service]
Type=oneshot
ExecStart=/usr/local/bin/say-hello.sh
```

```ini
# /etc/systemd/system/say-hello.timer
[Unit]
Description=Run say-hello every 2 minutes

[Timer]
OnBootSec=1min            # first run, 1 min after boot
OnUnitActiveSec=2min      # then every 2 min after each run
Persistent=true           # catch up missed runs after sleep/reboot

[Install]
WantedBy=timers.target
```

Or, calendar style:

```ini
[Timer]
OnCalendar=*-*-* *:00,30:00   # every 30 minutes, on the hour and half hour
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now say-hello.timer
systemctl list-timers              # all timers + next firing
```

> The trick: you enable the **.timer**, not the **.service**. The timer is what runs at boot; it triggers the service.

---

## Exercise 4 — Override a unit

```bash
sudo systemctl edit ssh
```

This opens an empty editor — anything you write goes to `/etc/systemd/system/ssh.service.d/override.conf`. Don't repeat the whole unit; just the bits you want to change:

```ini
[Service]
RestartSec=5
```

Save → systemd auto-reloads → `systemctl daemon-reload` is not needed (edit handles it). `systemctl cat ssh` will show the original PLUS your override at the bottom.

Reset an override:

```bash
sudo systemctl revert ssh
```

**Why drop-ins, not /etc edits?** Package updates overwrite `/lib/systemd/system/ssh.service`. Drop-ins under `/etc/systemd/system/ssh.service.d/` survive.

---

## Exercise 5 — journalctl mastery

```bash
journalctl                              # everything
journalctl -u ssh                       # specific unit
journalctl -u ssh -f                    # follow
journalctl --since "1 hour ago"
journalctl --since today --until "1 hour ago"
journalctl -p err                       # priority: emerg|alert|crit|err|warning|notice|info|debug
journalctl -b           # this boot
journalctl -b -1        # previous
journalctl --disk-usage
journalctl _COMM=sshd                   # by process name
journalctl _PID=1234                    # by pid
journalctl -o json-pretty               # structured fields revealed
```

> Each journal entry has structured fields (`_PID`, `_UID`, `_SYSTEMD_UNIT`, `MESSAGE`, …). That's why filtering is so powerful compared to grepping a text file.

Limit journal size:

```ini
# /etc/systemd/journald.conf
SystemMaxUse=500M
MaxRetentionSec=1month
```

---

## Exercise 6 — Cleanup

```bash
sudo systemctl disable --now heartbeat
sudo systemctl disable --now say-hello.timer
sudo rm /etc/systemd/system/heartbeat.service
sudo rm /etc/systemd/system/say-hello.service
sudo rm /etc/systemd/system/say-hello.timer
sudo systemctl daemon-reload
```

If you forget the daemon-reload, systemd still thinks the units exist — confusing.

---

## systemd service > `nohup script.sh &`

1. **Automatic restart** — `Restart=on-failure` brings it back. nohup doesn't.
2. **Survives reboot** — `enable` makes it persistent; nohup dies with the host.
3. **Logging** — journald captures stdout/stderr automatically. nohup dumps everything into `nohup.out` in your home dir.
4. **Dependencies** — `After=network-online.target` waits for prerequisites. nohup runs whenever.
5. **Resource limits** — `MemoryMax=`, `CPUQuota=`, `TasksMax=` enforced by cgroups. nohup, no.
6. **Privilege drop** — `User=`, `Group=`, `NoNewPrivileges=`, `ProtectSystem=strict` sandbox the service. nohup runs as you.
7. **Clean shutdown** — `systemctl stop` sends SIGTERM and waits; nohup leaves you to `kill` it yourself.

---

## Hardening cheatsheet for your own services

```ini
[Service]
User=svcuser
Group=svcuser
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict        # /usr, /boot read-only
ProtectHome=true
ReadWritePaths=/var/lib/myapp
RestrictAddressFamilies=AF_INET AF_INET6
SystemCallFilter=@system-service
MemoryMax=512M
```

Every one of these limits blast-radius if the service is compromised. Run `systemd-analyze security myservice` for a 0–10 exposure score.

---

## Common pitfalls

- **Forgetting `daemon-reload`** after editing a unit. Symptom: changes appear to do nothing.
- **`WantedBy=` missing** → `enable` says "no installation info; unit may be statically enabled or alias". Service runs only when started manually.
- **Wrong `Type=`** → systemd thinks the service died as soon as it forked. Common for daemons; use `Type=forking` (with `PIDFile=`) or refactor to run in foreground (`Type=simple`).
- **`ExecStart=/path/to/cmd && /other`** doesn't work — Exec lines aren't shell, no `&&`, `|`, etc. Wrap in `bash -c "…"` or split into multiple `ExecStartPre=`.
- **Editing `/lib/systemd/system/…`** — gets overwritten by apt. Always `systemctl edit` (drop-in).

→ [Module 13](../../module-13-logging-and-monitoring/README.md)
