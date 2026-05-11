# Module 12 — Exercises

## Exercise 1 — Write your first service

Create `/usr/local/bin/heartbeat.sh`:

```bash
#!/bin/bash
while true; do
  echo "alive at $(date)"
  sleep 10
done
```

Make it executable: `sudo chmod +x /usr/local/bin/heartbeat.sh`.

Create `/etc/systemd/system/heartbeat.service`:

```ini
[Unit]
Description=A heartbeat that prints to journal
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/heartbeat.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Activate:

```bash
sudo systemctl daemon-reload
sudo systemctl start heartbeat
sudo systemctl status heartbeat
sudo journalctl -u heartbeat -f       # follow logs
```

Then to make it survive reboot:

```bash
sudo systemctl enable heartbeat
```

## Exercise 2 — Stop, restart, disable

```bash
sudo systemctl stop heartbeat
sudo systemctl restart heartbeat
sudo systemctl disable heartbeat       # won't start on boot anymore
sudo systemctl is-enabled heartbeat
sudo systemctl is-active heartbeat
```

## Exercise 3 — Timers

Create `/usr/local/bin/say-hello.sh`:

```bash
#!/bin/bash
echo "hello at $(date)"
```

Service unit `/etc/systemd/system/say-hello.service`:

```ini
[Unit]
Description=Says hello

[Service]
Type=oneshot
ExecStart=/usr/local/bin/say-hello.sh
```

Timer unit `/etc/systemd/system/say-hello.timer`:

```ini
[Unit]
Description=Run say-hello every 2 minutes

[Timer]
OnBootSec=1min
OnUnitActiveSec=2min

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now say-hello.timer
systemctl list-timers
journalctl -u say-hello -f
```

## Exercise 4 — Override an existing service

Instead of editing `/lib/systemd/system/ssh.service` (which gets overwritten on update), do:

```bash
sudo systemctl edit ssh
```

This opens an editor for a *drop-in override* in `/etc/systemd/system/ssh.service.d/override.conf`. Add (for example):

```ini
[Service]
RestartSec=5
```

Save. Run `systemctl daemon-reload` then `systemctl restart ssh`.

## Exercise 5 — Journalctl mastery

```bash
journalctl                                # all logs
journalctl -u ssh                         # only ssh
journalctl -u ssh -f                      # follow (like tail -f)
journalctl --since "1 hour ago"
journalctl --since today --until "1 hour ago"
journalctl -p err                         # only errors and worse
journalctl -b                             # this boot
journalctl -b -1                          # previous boot
journalctl --disk-usage                   # how much space?
```

Note: journalctl is paginated with `less`. Search with `/`, quit with `q`.

## Exercise 6 — Cleanup

```bash
sudo systemctl disable --now heartbeat
sudo systemctl disable --now say-hello.timer
sudo rm /etc/systemd/system/heartbeat.service
sudo rm /etc/systemd/system/say-hello.service
sudo rm /etc/systemd/system/say-hello.timer
sudo systemctl daemon-reload
```

## Reflection

What's the advantage of a systemd service over `nohup script.sh &`? List at least three.
