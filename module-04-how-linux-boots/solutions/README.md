# Module 04 — Solutions

## Exercise 1 — Read your boot

```bash
dmesg | less
journalctl -b
journalctl -b -1
```

**What you should see, roughly in order:**

1. `Linux version …` — kernel version, compiler, build date
2. `Command line: …` — what GRUB passed to the kernel
3. CPU detection (`smpboot: CPU0 …`)
4. Memory map (`BIOS-e820: …`)
5. ACPI / device probing
6. Block-device discovery (`sda`, `nvme0n1` …)
7. Filesystem mounted as root
8. `systemd[1]: Starting …` — user space takes over
9. Network coming up
10. `Started Login Service.`

> 🧠 `dmesg` is just the **kernel ring buffer**. `journalctl -b` is the **systemd journal** for the current boot — kernel + every unit. `-b -1` is the previous boot, `-b -2` two boots ago, etc.

---

## Exercise 2 — systemd inventory

```bash
systemctl list-units --type=service                   # everything systemd knows about
systemctl list-units --type=service --state=running   # only running
systemctl status ssh
systemctl cat ssh
```

`systemctl cat` shows the actual unit file **plus** any drop-in overrides — much safer than `cat /lib/systemd/system/…` directly, because it tells you which files are in play.

Anatomy of a unit:

```ini
[Unit]      → metadata + dependencies (Description, After=, Requires=)
[Service]   → how to run the program (ExecStart=, User=, Restart=)
[Install]   → what `systemctl enable` hooks into (WantedBy=…)
```

---

## Exercise 3 — Targets

```bash
systemctl get-default
# typical answers:
#   graphical.target     (desktop)
#   multi-user.target    (server, no GUI)
```

`systemctl list-units --type=target` shows every target currently *active*. The dependency graph is what brings them up:

```
basic.target → multi-user.target → graphical.target
```

Change the default (don't do this casually):

```bash
sudo systemctl set-default multi-user.target
```

---

## Exercise 4 — Boot timing

```bash
systemd-analyze
# Startup finished in 1.2s (kernel) + 4.1s (userspace) = 5.3s
systemd-analyze blame          # slowest units, descending
systemd-analyze critical-chain # the actual blocking path to graphical.target
```

`blame` is *what took the longest*. `critical-chain` is *what kept boot waiting*. They're not the same — a slow unit that nothing depends on doesn't actually delay your login prompt.

Common slow culprits: `NetworkManager-wait-online.service`, `snapd.seeded.service`, `plymouth-quit-wait.service`.

---

## Exercise 5 — GRUB peek

When you pressed `e`, you saw lines like:

```
linux  /boot/vmlinuz-6.5.0-…  root=UUID=… ro quiet splash
initrd /boot/initrd.img-6.5.0-…
```

- `linux` — the kernel image
- `root=UUID=…` — which filesystem to use as `/`
- `ro` — mount it read-only first (fsck-able), remount rw later
- `quiet splash` — hide boot messages (remove these to debug; you'll see all kernel messages on screen)
- `initrd` — the initramfs

**Esc** backs out without saving — nothing was persisted.

---

## Exercise 6 — Single-user rescue

Adding `single` (or `init=/bin/bash`, or `systemd.unit=rescue.target`) drops you to a minimal root shell **before** most services start. This is your "the system is broken, I need to fix it" mode.

Once in the shell, the root filesystem is often read-only. Make it writable:

```bash
mount -o remount,rw /
```

Typical uses:
- Reset a forgotten root password (`passwd`)
- Fix a broken `/etc/fstab` that hangs boot
- Disable a service that's crash-looping

Reboot back to normal: `exec /sbin/init` or `reboot -f`.

---

## Exercise 7 — Find the bootloader config

```bash
ls /boot/
# vmlinuz-*        → kernel images
# initrd.img-*     → initramfs
# config-*         → kernel build config
# System.map-*     → symbol table (useful for debugging)
# grub/            → GRUB config + modules

cat /etc/default/grub      # high-level knobs — what you edit
ls /boot/grub/             # grub.cfg is the *generated* config — never edit by hand
```

The pipeline is:

```
edit /etc/default/grub   →   sudo update-grub   →   /boot/grub/grub.cfg
```

Editing `grub.cfg` directly works once but gets overwritten on the next kernel update.

---

## Common pitfalls

- **Hand-editing `grub.cfg`** instead of `/etc/default/grub` + `update-grub`.
- **Forgetting `systemctl daemon-reload`** after editing a unit file.
- **Identifying disks by `/dev/sdb1`** in fstab — these can shuffle. Use UUID.
- **Removing `quiet splash`** is great for debugging but ugly for daily use; keep one extra GRUB entry with verbose flags if you want both.

→ [Module 05](../../module-05-shell-scripting-basics/README.md)
