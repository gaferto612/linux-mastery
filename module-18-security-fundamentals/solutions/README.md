# Module 18 — Solutions

## Exercise 1 — Harden SSH

Edit `/etc/ssh/sshd_config` (backup first: `sudo cp /etc/ssh/sshd_config{,.bak}`):

```ini
PermitRootLogin no                # never log in as root over SSH
PasswordAuthentication no         # keys only
PubkeyAuthentication yes
KbdInteractiveAuthentication no
ChallengeResponseAuthentication no
UsePAM yes
Port 2222                         # optional — reduces drive-by scan noise
AllowUsers yourusername           # whitelist of allowed accounts
MaxAuthTries 3
LoginGraceTime 30
ClientAliveInterval 300
ClientAliveCountMax 2

# Optional, very strict ciphers (auto-rejects ancient clients):
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
```

**Always test before restart:**

```bash
sudo sshd -t                                      # syntax check
# Open a SECOND SSH session and leave it logged in. Then:
sudo systemctl restart ssh
# In a third terminal, confirm new auth works.
# Only close the safety session after the third works.
```

`PermitRootLogin no` is the highest-impact single change — most automated attacks target `root`.

Non-standard `Port 2222` is **obscurity, not security**. It cuts log noise but doesn't slow a targeted attacker.

---

## Exercise 2 — fail2ban

```bash
sudo apt install fail2ban
sudo systemctl enable --now fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

How it works: fail2ban watches `/var/log/auth.log` (or journal), counts failures matching the `sshd` filter, and when a source IP exceeds `maxretry` within `findtime` it writes an iptables/nftables rule banning the IP for `bantime`.

Customize in `/etc/fail2ban/jail.local` (NEVER edit `jail.conf` — that's overwritten on update):

```ini
[DEFAULT]
bantime  = 1h
findtime = 10m
maxretry = 5

[sshd]
enabled = true
port    = 2222          # if you changed it
```

Inspect a ban:

```bash
sudo fail2ban-client status sshd
# Banned IP list: 1.2.3.4 5.6.7.8
sudo fail2ban-client unban 1.2.3.4
```

---

## Exercise 3 — ufw policy

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2222/tcp comment 'SSH on custom port'
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status verbose
```

> ⚠️ **Always allow your SSH port BEFORE `ufw enable` on a remote box.** Otherwise you instantly lock yourself out.

ufw is a friendly frontend; under the hood it writes nftables/iptables rules. `sudo iptables -L -n -v` (or `nft list ruleset`) shows them. Useful when debugging "but ufw says it's allowed".

For larger deployments, the right answer is `nftables` directly (modern) or configuration via Ansible / Terraform.

---

## Exercise 4 — Password policy

`/etc/login.defs` (system-wide aging defaults applied to NEW users):

```
PASS_MAX_DAYS   90
PASS_MIN_DAYS   7
PASS_WARN_AGE   14
```

For EXISTING users:

```bash
sudo chage -M 90 -m 7 -W 14 alice
sudo chage -l alice
```

Strength enforcement via PAM:

```bash
sudo apt install libpam-pwquality
```

`/etc/security/pwquality.conf`:

```
minlen = 12
minclass = 3                  # at least 3 of: lower/upper/digit/special
maxrepeat = 3                 # no run of 3+ same char
dictcheck = 1                 # reject dictionary words
```

Modern thinking (NIST 800-63B): length > complexity. A 16-char passphrase beats `P@ssw0rd!`. Don't force frequent rotation if you have MFA and breach detection — rotation drives users to write passwords down.

---

## Exercise 5 — Disable what you don't need

```bash
systemctl list-units --type=service --state=running
```

For each running service, ask: *do I actually use this?* Common things you can disable on a server:

```bash
sudo systemctl disable --now cups          # printing
sudo systemctl disable --now avahi-daemon  # zero-conf networking
sudo systemctl disable --now bluetooth
sudo systemctl disable --now ModemManager
```

**Don't disable services you don't recognize.** Look them up first: `systemctl cat <name>` shows the unit's Description and command line.

Minimal exposure = smaller attack surface. Every listening port and running daemon is a thing that can be exploited.

---

## Exercise 6 — Lynis audit

```bash
sudo apt install lynis
sudo lynis audit system
```

Lynis runs ~200 checks and produces a hardening index. The output sections:

```
[ Hardening index : 65 / 100 ]
Warnings:          12
Suggestions:       42
```

Read the *Suggestions* — each has a test-id, a description, and a fix. Pick 5 you understand, apply them, re-run. Track your hardening index over time.

Logs are in `/var/log/lynis.log` and `/var/log/lynis-report.dat`.

Other auditors worth knowing: **OpenSCAP** (compliance frameworks: CIS, STIG), **debsecan** (CVEs in installed packages), **chkrootkit** / **rkhunter** (rootkit scanning).

---

## Exercise 7 — Find SUID binaries

```bash
find / -perm -4000 -type f 2>/dev/null
```

SUID = "set user ID on execution" = "this binary runs as its OWNER, regardless of who runs it". Required for things like:

- `/usr/bin/sudo`, `/usr/bin/su` — must elevate to root
- `/usr/bin/passwd` — needs to write `/etc/shadow`
- `/usr/bin/mount`, `/usr/bin/ping` — historically needed root for raw sockets

Anything you don't recognize is an investigation target. SUID = privilege escalation vector if the binary has a bug.

Modern Linux often replaces SUID with **file capabilities** (`getcap` / `setcap`). `/usr/bin/ping` no longer needs SUID — it has `cap_net_raw`. Capability-based privilege is more granular and safer.

```bash
getcap -r /usr/bin 2>/dev/null    # list binaries with capabilities
```

---

## Exercise 8 — Bash history hygiene

```bash
# ~/.bashrc
export HISTCONTROL=ignoreboth:erasedups
export HISTIGNORE='ls:cd:pwd:exit:clear:history:* --password=*:* --token=*'
export HISTSIZE=10000
export HISTFILESIZE=20000
shopt -s histappend                   # don't overwrite history on shell exit
PROMPT_COMMAND='history -a'           # write history after every command
```

Better: **prefix any one-off command containing a secret with a space**. With `HISTCONTROL=ignorespace`, those lines are never recorded.

Even better: don't put secrets on the command line at all. Use env vars from a file, `pass`, `gpg`, or a secret manager.

```bash
export DB_PASS=$(pass show db/prod)      # never typed; never in history
```

---

## Threat-model framing

When deciding what to harden, think:

1. **Who?** Script kiddies, opportunistic worms, targeted attackers, insider, supply chain.
2. **What are they after?** Crypto-mining CPU, data, persistence, lateral movement.
3. **How do they get in?** SSH brute force, web app vuln, supply chain, phishing.
4. **What's the blast radius if they succeed?** Single user? Full system? Lateral access to other hosts?

A VPS running a personal blog has very different priorities than a corporate fileserver.

---

## What Ubuntu gets wrong by default

The honest top 5:

1. **Password SSH auth enabled.** Should be off the moment you have a key.
2. **`root` login over SSH defaults vary**, but `PermitRootLogin prohibit-password` still allows key-based root SSH. Set to `no`.
3. **No firewall by default.** ufw is installed but inactive.
4. **No automatic security updates** out of the box (though `unattended-upgrades` is one apt install away).
5. **History contains your secrets.** No `HISTCONTROL` defaults to filtering anything.

Bonus: snapd, telemetry, motd-news fetches — all worth reviewing for a hardened box.

---

## Common pitfalls

- **Restarting sshd without a second session open** → permanent lockout if config breaks.
- **`ufw enable` without explicitly allowing SSH first** → lockout.
- **Locking down too hard** → users work around the controls (shared keys, side channels). Security that gets bypassed is worse than no security.
- **Trusting password complexity over MFA.** 2FA stops most credential theft regardless of password.
- **Logging everything, monitoring nothing.** Logs you never read don't protect you. Set up alerts.
- **Following hardening guides blindly.** Some recommendations come from compliance frameworks designed for different threat models. Understand each change before applying.

→ [Module 19](../../module-19-intro-to-pentesting/README.md)
