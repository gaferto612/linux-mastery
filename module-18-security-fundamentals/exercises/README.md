# Module 18 — Exercises

## Exercise 1 — Harden SSH

Edit `/etc/ssh/sshd_config` (back it up first):

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
Port 2222                     # optional, but reduces noise from scanners
AllowUsers yourusername       # whitelist
```

**Before restarting SSH, make sure you have key auth working in another terminal — or you'll lock yourself out.**

```bash
sudo sshd -t                  # test syntax
sudo systemctl restart ssh
```

In another terminal: confirm you can still log in.

## Exercise 2 — fail2ban

```bash
sudo apt install fail2ban
sudo systemctl enable --now fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

fail2ban watches logs and bans IPs that fail too many times. Defaults are sensible.

## Exercise 3 — ufw policy

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2222/tcp        # SSH on your custom port
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status verbose
```

## Exercise 4 — Password policy

Look at `/etc/login.defs`. Set:

```
PASS_MAX_DAYS   90
PASS_MIN_DAYS   7
PASS_WARN_AGE   14
```

Install `libpam-pwquality`:

```bash
sudo apt install libpam-pwquality
```

Look at `/etc/security/pwquality.conf`. Configure minimum length, complexity.

## Exercise 5 — Disable what you don't need

```bash
systemctl list-units --type=service --state=running
```

For each running service, ask: do I actually use it? Disable what you don't:

```bash
sudo systemctl disable --now servicename
```

(Don't disable things you don't recognize. Look them up first.)

## Exercise 6 — Lynis audit

```bash
sudo apt install lynis
sudo lynis audit system
```

Lynis runs 200+ checks and gives you a hardening score with suggestions. Read the report. Pick 5 suggestions and apply them.

## Exercise 7 — Find SUID binaries

```bash
find / -perm -4000 -type f 2>/dev/null
```

These run as their owner (often root) regardless of who runs them. They're potential attack surface. Most should look familiar (`sudo`, `passwd`, `su`). Anything you don't recognize is worth investigating.

## Exercise 8 — Bash history hygiene

Add to `~/.bashrc`:

```bash
export HISTCONTROL=ignoreboth:erasedups
export HISTIGNORE='ls:cd:pwd:exit:clear:history:* --password=*:* --token=*'
```

This stops sensitive commands from being recorded.

Better: when running a one-off command with a secret, prefix with a space. Lines starting with space are not saved (`HISTCONTROL=ignorespace`).

## Reflection

What does a fresh Ubuntu installation get wrong by default, from a security perspective? List your top 5 hardening priorities.
