# Module 14 — Exercises

## Exercise 1 — rsync fundamentals

```bash
mkdir -p ~/src ~/dest
echo "hello" > ~/src/file1.txt
echo "world" > ~/src/file2.txt

rsync -av ~/src/ ~/dest/      # note the trailing slash on src!
```

The trailing slash on the source matters. Try without it and see the difference.

Useful flags:
- `-a` (archive) = `-rlptgoD` (recursive, links, perms, times, group, owner, devices)
- `-v` verbose
- `-n` dry run (always try this first)
- `--delete` make destination match source exactly (DANGEROUS if you mistype paths)
- `--progress` show progress for big files

## Exercise 2 — rsync over SSH

```bash
rsync -av ~/src/ user@server:/path/to/backup/
```

That's it. SSH transport is automatic. Now you have working remote backups.

## Exercise 3 — Cron job

```bash
crontab -e
```

Add a line:

```
*/5 * * * * /usr/local/bin/say-hello.sh >> /tmp/cron.log 2>&1
```

This runs every 5 minutes. Watch `/tmp/cron.log` grow.

Common cron patterns:
- `0 * * * *` — every hour, on the hour
- `0 2 * * *` — 2:00 AM every day
- `0 2 * * 0` — 2:00 AM every Sunday
- `*/15 * * * *` — every 15 minutes

Remove the cron with `crontab -e` again.

## Exercise 4 — Same thing with systemd timer

Re-do exercise 3 using a systemd timer (review Module 12 if needed). Compare the experience.

## Exercise 5 — restic (real backups)

```bash
sudo apt install restic
mkdir -p ~/restic-repo
restic init --repo ~/restic-repo       # set a password
restic backup --repo ~/restic-repo ~/Documents
restic snapshots --repo ~/restic-repo
```

To restore:

```bash
mkdir /tmp/restore
restic restore latest --repo ~/restic-repo --target /tmp/restore
ls /tmp/restore
```

**This is the most important step in this whole module.** A backup you've never restored from is hope, not insurance.

## Exercise 6 — Build a backup script

Write `backup.sh`:

```bash
#!/bin/bash
set -euo pipefail
export RESTIC_PASSWORD_FILE=/root/.restic-pwd
restic backup --repo /backup/restic-repo /etc /home /var/lib
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --prune
```

Schedule it via systemd timer to run daily at 3 AM. Email yourself if it fails (`OnFailure=` in the unit).

## Exercise 7 — Automate something annoying

Think of one tedious thing you do regularly. Write a script for it. Schedule it. Examples:
- Clean `/tmp` of files older than 7 days
- Email yourself a daily report of disk usage
- Rotate your own application logs

## Reflection

If your VM died right now and you had to rebuild it on new hardware, what would you lose? What would still be in your backups? **Map this out.** That's your real backup audit.
