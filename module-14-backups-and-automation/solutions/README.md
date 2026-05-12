# Module 14 — Solutions

## Exercise 1 — rsync fundamentals

```bash
rsync -av ~/src/ ~/dest/        # WITH trailing slash on src
rsync -av ~/src  ~/dest/        # WITHOUT — copies the src DIRECTORY into dest
```

The trailing-slash rule:

```
src/   →  copies contents of src into dest/
src    →  copies src itself (creating dest/src/…)
```

This trips up everyone exactly once. Get burnt, learn it forever.

Useful flag combinations:

```bash
rsync -avn ~/src/ ~/dest/             # -n DRY RUN — always do this first
rsync -av --progress ~/src/ ~/dest/   # progress bars on big files
rsync -av --delete ~/src/ ~/dest/     # make dest IDENTICAL to src (deletes extras)
rsync -av --exclude='*.tmp' …
rsync -av --link-dest=/backup/prev/ ~/src/ /backup/today/   # incremental via hard links
```

`--delete` + a typo in the source path is the rsync equivalent of `rm -rf /` — practice with `-n` first.

---

## Exercise 2 — rsync over SSH

```bash
rsync -av ~/src/ user@server:/path/backup/
rsync -av user@server:/etc/ ~/etc-snapshot/   # pull
rsync -av -e 'ssh -i ~/.ssh/backup_key -p 2222' ~/src/ user@server:/path/
```

rsync auto-detects the `user@host:` form and uses SSH. `-e` overrides the SSH command for keys/ports.

The remote needs `rsync` installed too — both sides talk the rsync protocol over the SSH pipe.

---

## Exercise 3 — Cron

```bash
crontab -e
```

Add:

```
*/5 * * * * /usr/local/bin/say-hello.sh >> /tmp/cron.log 2>&1
```

Cron syntax:

```
┌── min     0-59
│ ┌── hour  0-23
│ │ ┌── day-of-month  1-31
│ │ │ ┌── month       1-12
│ │ │ │ ┌── day-of-week 0-6 (0 = Sun, 7 also = Sun)
│ │ │ │ │
* * * * *  /path/to/cmd
```

Useful shorthands cron understands: `@reboot`, `@daily` (= `0 0 * * *`), `@hourly`, `@weekly`.

**Why `>> /tmp/cron.log 2>&1`** — cron's default behavior is to email stdout+stderr to the local user. If you have no MTA, you get nothing and your job silently fails. Always redirect.

**Why your `$PATH` is suddenly empty in cron** — cron runs with a minimal environment. Use absolute paths (`/usr/bin/python3`, not `python3`) or set `PATH=...` at the top of the crontab.

Remove with `crontab -e` and delete the line, or `crontab -r` to clear everything (careful).

---

## Exercise 4 — Same thing with systemd timer

See Module 12 Exercise 3. The same job:

```ini
# /etc/systemd/system/say-hello.timer
[Timer]
OnCalendar=*:0/5            # every 5 minutes
Persistent=true
```

Why timer ≥ cron for new work:

- Output goes to journald automatically — no `2>&1` boilerplate.
- `Persistent=true` runs missed jobs after the machine wakes up (cron just skips).
- Dependencies (`After=network-online.target`) — cron has none.
- Failures are inspectable with `systemctl status say-hello.service`.
- Resource limits via cgroups.

cron's win: ubiquitous, two-line syntax, works in a Docker container or BusyBox box where systemd isn't.

---

## Exercise 5 — restic (real backups)

```bash
sudo apt install restic
mkdir -p ~/restic-repo
restic init --repo ~/restic-repo                  # prompts for password
restic backup --repo ~/restic-repo ~/Documents
restic snapshots --repo ~/restic-repo             # list all snapshots
restic stats --repo ~/restic-repo                 # see dedup savings
```

Restore:

```bash
mkdir /tmp/restore
restic restore latest --repo ~/restic-repo --target /tmp/restore
ls /tmp/restore                                   # full /home/you/Documents tree under here
```

What restic does that tar doesn't:

- **Content-addressed dedup** — same file in 100 snapshots = stored once.
- **Encryption by default** — repo password unlocks all snapshots.
- **Incremental forever** — first snapshot full, each subsequent only changed chunks.
- **Browse-able** — `restic mount /mnt` mounts the whole repo as a FUSE filesystem.

> 🔥 **An untested backup is not a backup.** The restore step is the whole point. Schedule a quarterly drill: pick a random file from last week, restore it, diff it against current. If that fails, you find out NOW, not during an incident.

---

## Exercise 6 — Build a backup script

```bash
#!/bin/bash
set -euo pipefail
export RESTIC_PASSWORD_FILE=/root/.restic-pwd     # mode 600, owned by root
export RESTIC_REPOSITORY=/backup/restic-repo
export HOME=/root                                  # ensure HOME for restic cache

restic backup --tag daily /etc /home /var/lib
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --prune
restic check --read-data-subset=5%                 # verify a sample of data
```

`forget --prune` is the GFS retention dance: keep 7 dailies, 4 weeklies, 6 monthlies, then garbage-collect chunks no snapshot references.

Schedule + notify on failure:

```ini
# /etc/systemd/system/backup.service
[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh

[Unit]
OnFailure=backup-failed.service     # separate service that emails / pings you
```

```ini
# /etc/systemd/system/backup.timer
[Timer]
OnCalendar=daily
RandomizedDelaySec=30min            # don't all hammer the repo at once if you run many
Persistent=true

[Install]
WantedBy=timers.target
```

---

## Exercise 7 — Automate something annoying

Ideas with real payoff:

```bash
# Clean /tmp of files older than 7 days
find /tmp -type f -mtime +7 -delete

# Daily disk-usage report
df -h | mail -s "[$(hostname)] disk report" you@example.com

# Rotate your own app log
mv /var/log/myapp.log /var/log/myapp.log.$(date +%F)
systemctl reload myapp

# Sync laptop to NAS while you sleep
rsync -av --delete --exclude=node_modules ~/projects/ nas:/backups/laptop/projects/
```

The principle: any task you do more than twice a year, write it down. Twice a month → script it. Twice a week → schedule it.

---

## "If my VM died right now" audit

Walk yourself through:

1. What's on it that's **not in git**? (configs in `/etc`, secrets, local databases)
2. What's in git but **not pushed**?
3. What's in restic but **never restored**?
4. What would you have to **rebuild from memory**? (cron entries, custom scripts in `/usr/local/bin`, firewall rules)

That gap analysis is the backup audit. Close the gaps before you need them.

---

## Common pitfalls

- **Untested backups.** Hope, not insurance.
- **Backing up the database files of a running DB.** You get a corrupt snapshot. Use `pg_dump` / `mysqldump` / engine-specific tools, then back up the dump.
- **Forgetting `--exclude=node_modules`** etc. — your backups are 10× bigger and rebuildable junk dominates.
- **Single-key encryption with no break-glass copy.** Lose the restic password → all backups gone.
- **Backups stored on the same machine.** Disk fails → both gone. The "off-site" leg matters.
- **`rsync --delete` with a typo on the source path** — wipes everything in dest. Always `-n` first.

→ [Module 15](../../module-15-systems-programming-intro/README.md)
