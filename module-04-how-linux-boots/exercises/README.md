# Module 04 — Exercises

## Exercise 1 — Read your boot

```bash
dmesg | less                    # kernel ring buffer
journalctl -b                   # current boot's logs via systemd journal
journalctl -b -1                # previous boot
```

Scroll. Don't try to understand everything. Look for patterns: "Linux version", driver messages, mounted filesystems.

## Exercise 2 — systemd inventory

```bash
systemctl list-units --type=service
systemctl list-units --type=service --state=running
systemctl status ssh         # or any service that exists on your system
```

Find the *file* behind a service:

```bash
systemctl cat ssh
```

Read it. Note the `[Unit]`, `[Service]`, `[Install]` sections.

## Exercise 3 — Targets (think: runlevels)

```bash
systemctl get-default        # what target boots by default?
systemctl list-units --type=target
```

(Don't change the default unless you know what you're doing.)

## Exercise 4 — Boot timing

```bash
systemd-analyze
systemd-analyze blame        # what took the longest to start?
systemd-analyze critical-chain
```

What's slow on your machine? Anything surprising?

## Exercise 5 — GRUB peek

On your VM, reboot. When the GRUB menu appears, press **`e`** to edit the boot entry. Look at it. Don't change anything yet — just observe the kernel command line.

(Press Esc to back out without saving.)

## Exercise 6 — Single-user rescue (VM only!)

In GRUB, edit the boot entry. Find the line starting with `linux`. Add ` single` (or `init=/bin/bash`) at the end. Boot with Ctrl-X.

You should land in a minimal root shell. This is how you fix systems with broken configs or forgotten passwords.

To exit: `reboot`.

**Only do this on a VM you can afford to break.**

## Exercise 7 — Find the bootloader config

```bash
ls /boot/
cat /etc/default/grub
ls /boot/grub/   # or /boot/grub2/
```

Don't modify these yet. Just look.

## Reflection

Write in your notes: in your own words, the sequence from pressing the power button to seeing the login prompt. ~10 steps. Don't look it up.
