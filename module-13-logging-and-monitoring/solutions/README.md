# Module 13 — Solutions

## Exercise 1 — Map /var/log

```bash
ls -lh /var/log/
```

Typical Debian/Ubuntu inventory:

```
auth.log       login + sudo + ssh
syslog         general "everything" via rsyslog
kern.log       kernel ring buffer copy
dpkg.log       package operations (install/remove/upgrade)
apt/           apt's own logs (history.log, term.log)
journal/       binary systemd journal (per-boot)
nginx/         per-service log dir (if installed)
```

RHEL/CentOS differs: `/var/log/messages` (≈ syslog), `/var/log/secure` (≈ auth.log), `/var/log/maillog`. Same idea, different names.

`auth.log` is gold for incident response:

```bash
sudo grep "Failed password" /var/log/auth.log | tail
sudo grep "Accepted publickey" /var/log/auth.log | tail
sudo grep sudo /var/log/auth.log | tail
```

---

## Exercise 2 — journalctl vs files

```bash
sudo grep "Failed password" /var/log/auth.log | tail
sudo journalctl _COMM=sshd | grep "Failed password" | tail
```

Both work. The difference:

| Source             | Format    | Filter          | Rotation              |
|--------------------|-----------|-----------------|------------------------|
| `/var/log/*.log`   | text      | grep            | logrotate (cron/timer) |
| journald binary    | structured| `journalctl -p`/`_FIELD=` | journald (built-in) |

On modern systems journald gets the events FIRST; rsyslog reads them out of the journal and writes the text files. So both are usually saying the same thing.

You can disable text logs entirely if you commit to journalctl (`sudo systemctl disable --now rsyslog`).

---

## Exercise 3 — Configure logrotate

```
# /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily          # rotate every day
    rotate 7       # keep 7 rotated files
    compress       # gzip rotated files
    missingok      # OK if log doesn't exist
    notifempty     # don't rotate empty logs
    create 0640 root root   # permissions for fresh log after rotation
}
```

Test:

```bash
sudo logrotate -d /etc/logrotate.d/myapp        # debug, no action
sudo logrotate -f /etc/logrotate.d/myapp        # FORCE a rotation now
sudo logrotate -v /etc/logrotate.conf           # run the whole policy
```

State file: `/var/lib/logrotate/status` — last-rotated timestamps. If you "tested" by force-rotating and now the daily rotation thinks it just ran, fix this file.

Sigh-and-reload: many daemons keep their log file open across rotation. `postrotate { systemctl reload nginx; }` (or `kill -HUP`) makes them reopen the new file. Otherwise they keep writing to the deleted (still-open) inode.

---

## Exercise 4 — top / htop column safari

```
%CPU      percentage of one CPU (so 200% on a quad-core = 2 cores pinned)
%MEM      percentage of RAM used by this process's RES
VIRT      virtual size (address space — almost meaningless on its own)
RES       resident set size — actual physical RAM in use   ← THE REAL NUMBER
SHR       shared (with other processes, e.g. libc mapped once)
S         state: R(running), S(sleeping), D(uninterruptible / disk wait), Z(zombie), T(stopped)
PR/NI     priority, niceness
```

Load average (1, 5, 15 min): number of processes either running or waiting on disk I/O. On an N-core box, 1.00 per core is "fully busy". 3.00 on a 4-core box = healthy headroom. 8.00 on a 4-core box = stuff is queuing up.

> **`D` state surprise:** a process in uninterruptible sleep doesn't respond to ANY signal, including SIGKILL. It's stuck in the kernel waiting on I/O. If it doesn't clear, the underlying device is broken.

---

## Exercise 5 — vmstat pulse

```bash
vmstat 1
```

What to watch:

```
r > #cores    → CPU saturated; processes queuing
b > 0         → blocked on I/O (slow disk? network FS?)
si/so > 0     → swapping! probably out of RAM
wa high       → CPU is idle but waiting on disk
us + sy + wa + id = 100 always
```

A healthy idle box: `r=0`, `b=0`, `si=0`, `so=0`, `id≈100`.

Generate load: `yes > /dev/null` pegs one core in user space (`us` ↑). Kill with `kill %1` or Ctrl-C if foreground.

---

## Exercise 6 — Disk I/O

```bash
sudo apt install iotop sysstat
sudo iotop                # interactive, per-process I/O
iostat -x 1               # per-disk extended stats, every second
```

`iostat -x` fields to know:

```
r/s, w/s         reads/writes per second
rkB/s, wkB/s     throughput
await            avg time (ms) a request spends in queue + service
%util            % of time the device was busy (100% = saturated)
```

Generate I/O and watch:

```bash
dd if=/dev/zero of=/tmp/bigfile bs=1M count=500
rm /tmp/bigfile
```

For SSDs, `%util` can be misleading — the device can service many ops in parallel and still show low %util while throughput is maxed. Look at `await` and queue depth too.

---

## Exercise 7 — Disk-full alert script

The one in the exercise is solid. The pieces explained:

```bash
df -P                       # POSIX output: same columns everywhere (portable)
tail -n +2                  # skip header line
read -r _fs _size _used _avail pct mount
                            # underscore = "I don't care about this field"
pct_num="${pct%\%}"         # parameter-expand: strip trailing %
```

The `df -P` choice matters: `df -h --output=…` is GNU-only. If your script needs to work on macOS, BSD, BusyBox, stick to POSIX flags.

Schedule via systemd timer (Module 12). Email/notify path options:

```bash
# email (mail package required)
echo "Disk warning" | mail -s "[host] disk >90%" you@example.com

# slack webhook
curl -sX POST -H 'Content-type: application/json' \
  --data '{"text":"[host] disk >90%"}' \
  "$SLACK_WEBHOOK_URL"
```

---

## "The server is slow" — first 5 commands

1. `uptime` — load average (instant sanity check)
2. `top` or `htop` — what's hot on CPU/memory right now
3. `vmstat 1 5` — system pulse, look for swap and `wa`
4. `iostat -x 1 5` — disk I/O saturation per device
5. `ss -s` or `iftop` — network connections / throughput
6. (bonus) `journalctl -p err --since "30 min ago"` — anything erroring lately

Each one is for a different dimension. You're not looking for a culprit yet — you're asking "which resource is the bottleneck?" Then dig in.

---

## Common pitfalls

- **Rotating without reload** — daemons keep writing to the old (deleted) inode. `postrotate { … reload … }` is essential.
- **Mistaking `VIRT` for memory in use** — modern programs map huge address spaces. RES is what matters.
- **Load average panic on slow disks** — load includes processes in `D` state. If 8 processes are stuck on a NAS, your load is 8 even though CPU is idle.
- **Logging at DEBUG in production** — fills disks, kills performance. Set log level appropriately.
- **No log rotation on your own apps.** Custom service writing to `/var/log/myapp.log` with no logrotate.d entry → eventually it eats the disk.

→ [Module 14](../../module-14-backups-and-automation/README.md)
