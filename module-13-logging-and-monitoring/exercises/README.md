# Module 13 — Exercises

## Exercise 1 — Map /var/log

```bash
ls -lh /var/log/
sudo less /var/log/syslog        # general (Debian/Ubuntu)
sudo less /var/log/auth.log      # login/auth attempts
sudo less /var/log/dpkg.log      # package operations
sudo less /var/log/kern.log      # kernel
```

Different distros differ. On RHEL, look in `/var/log/messages` and `/var/log/secure`.

## Exercise 2 — journalctl vs files

journalctl reads the binary journal; many tools also write to `/var/log` via syslog. Compare:

```bash
sudo grep "Failed password" /var/log/auth.log | tail
sudo journalctl _COMM=sshd | grep "Failed password" | tail
```

Same events, two sources.

## Exercise 3 — Configure logrotate

Create `/etc/logrotate.d/myapp`:

```
/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
    create 0640 root root
}
```

Test without waiting a day:

```bash
sudo logrotate -d /etc/logrotate.d/myapp     # debug (dry-run)
sudo logrotate -f /etc/logrotate.d/myapp     # force run
```

## Exercise 4 — top / htop column safari

Run `top` and identify:
- `%CPU`, `%MEM` — pretty obvious
- `RES` vs `VIRT` — resident vs virtual memory. Which is the "real" memory used?
- `S` — state. What's `D`? What's `Z`?
- Load average — what do the three numbers mean?

Read `man top` slowly over a few sessions.

## Exercise 5 — vmstat pulse

```bash
vmstat 1
```

A new line every second. Columns:
- `r` — processes waiting to run (queue depth)
- `b` — blocked on I/O
- `si`/`so` — swap in/out (any number here = your system is swapping; usually bad)
- `bi`/`bo` — block I/O in/out
- `us`/`sy`/`id`/`wa` — CPU breakdown: user, system, idle, I/O wait

Generate some load (`yes > /dev/null &`) and watch the numbers move. Kill it with `kill %1`.

## Exercise 6 — Disk I/O

Install: `sudo apt install iotop sysstat`.

```bash
sudo iotop                 # which process is hammering the disk?
iostat -x 1                # per-disk stats every second
```

In another terminal, generate I/O: `dd if=/dev/zero of=/tmp/bigfile bs=1M count=500`. Watch iotop. Delete the file after.

## Exercise 7 — Disk-full alert script

Write `disk-check.sh`:

```bash
#!/bin/bash
set -euo pipefail

THRESHOLD=90

# Skip the header line, then for each filesystem:
# field 5 is the use percentage, field 6 is the mount point.
df -P | tail -n +2 | while read -r _fs _size _used _avail pct mount; do
  pct_num="${pct%\%}"           # strip trailing %
  if [ "$pct_num" -ge "$THRESHOLD" ]; then
    echo "WARN: $mount at ${pct}"
  fi
done
```

Notes:
- We use `df -P` (POSIX output) rather than `df -h --output=...` because the `--output` flag is GNU-specific (won't work on BSD/macOS).
- The `while` loop runs in a subshell because of the pipe — that's fine here since we just print. If you wanted to *count* warnings and act on the total, you'd need a different pattern (process substitution `< <(df -P)` or readarray).

Schedule it via systemd timer (you learned this in Module 12) to run hourly.

Bonus: email yourself when triggered (`mail` command, or curl to a webhook).

## Reflection

A user says "the server is slow." What are the first 5 commands you'd run? (Hint: load avg, CPU, memory, disk I/O, network. One per dimension.)
