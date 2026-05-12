# Module 08 — Solutions

## Exercise 1 — Anatomy of /etc/passwd

```
root:x:0:0:root:/root:/bin/bash
 │   │ │ │  │      │       └── login shell
 │   │ │ │  │      └────────── home directory
 │   │ │ │  └───────────────── GECOS (full name / comment)
 │   │ │ └──────────────────── primary GID
 │   │ └────────────────────── UID
 │   └──────────────────────── password placeholder ("x" = look in /etc/shadow)
 └──────────────────────────── username
```

Why the password isn't here: `/etc/passwd` is **world-readable** (every process reads it for username↔UID lookups). Putting password hashes here would let any user run an offline crack against them. So Linux moved the hashes to `/etc/shadow` (root-only, mode 0640).

The `x` is just a marker meaning "real hash is in shadow". An empty field would mean no password required; a `*` or `!` means the account is locked.

---

## Exercise 2 — Create a user the proper way

```bash
sudo useradd -m -s /bin/bash alice
sudo passwd alice
```

- `-m` → create `/home/alice` from `/etc/skel`
- `-s /bin/bash` → otherwise you get whatever `useradd -D` defaults to (sometimes `/bin/sh`)
- `-U` → also create a same-name group (Ubuntu does this by default)

Switching shells:

```bash
su - alice         # the dash matters: full login shell, fresh env, cd to home
                   # without it, you stay in your dir with your env
whoami             # alice
pwd                # /home/alice
id                 # uid=1001(alice) gid=1001(alice) groups=1001(alice)
```

> **`adduser` (Debian)** is a friendly wrapper around `useradd`. Either works; `useradd` is the lower-level standard.

---

## Exercise 3 — Groups

```bash
sudo groupadd developers
sudo usermod -aG developers alice    # -a APPEND, -G supplementary groups
groups alice                         # alice : alice developers
id alice
```

**The footgun:** `sudo usermod -G developers alice` (without `-a`) **replaces** alice's supplementary groups with only `developers`. She'd lose `sudo`, `adm`, anything else. Always `-aG`.

Group membership is read at login time. Alice has to log out and back in (or `newgrp developers`) to see the new group in her session.

---

## Exercise 4 — /etc/shadow

```
alice:$6$saltsalt$hashhashhash…:19876:0:99999:7:::
  │      │                       │     │  │     │
  │      │                       │     │  │     └── warn days before expiry
  │      │                       │     │  └──────── max age (days, 99999=never)
  │      │                       │     └─────────── min age (days between changes)
  │      │                       └───────────────── last change (days since 1970-01-01)
  │      └───────────────────────────────────────── encrypted password
  └──────────────────────────────────────────────── username
```

Hash prefix tells you the algorithm:

```
$1$  →  MD5         (legacy, weak)
$2y$ →  bcrypt
$5$  →  SHA-256
$6$  →  SHA-512     (very common default)
$y$  →  yescrypt    (modern Debian default)
```

A leading `!` or `*` means the account is locked / has no password.

---

## Exercise 5 — sudoers

Always use `sudo visudo` — it syntax-checks before writing. Saving a broken sudoers file with vim/nano can lock you out of root.

```
alice ALL=(ALL) NOPASSWD: /usr/bin/apt update
   │   │   │      │            │
   │   │   │      │            └── exact command allowed
   │   │   │      └── no password required
   │   │   └── run as (ALL = any user, including root)
   │   └── on which hosts (ALL since we're not using a network sudoers)
   └── who
```

Testing:

```bash
su - alice
sudo apt update            # → runs, no prompt
sudo apt install vim       # → "Sorry, user alice is not allowed to execute '/usr/bin/apt install vim' as root"
```

> **Path matters.** `NOPASSWD: apt` (without `/usr/bin/`) is interpreted differently and is often unsafe. Always full paths in sudoers.

Better than editing `/etc/sudoers` directly: drop a file in `/etc/sudoers.d/`:

```bash
sudo visudo -f /etc/sudoers.d/alice-apt
```

---

## Exercise 6 — Lock and unlock

```bash
sudo passwd -l alice       # prepends ! to the hash in /etc/shadow
sudo passwd -S alice       # shows: alice L … (L = locked)
sudo passwd -u alice       # removes the !
```

Locking only blocks **password** logins. Key-based SSH would still work unless you also set `usermod -L` *and* set the shell to `/sbin/nologin`. To fully disable an account:

```bash
sudo usermod -L -e 1 -s /usr/sbin/nologin alice
```

(`-e 1` sets the expiry date to 1970-01-02 = expired.)

---

## Exercise 7 — Cleanup

```bash
sudo userdel -r alice      # -r removes home dir + mail spool
sudo groupdel developers
```

If alice still has running processes, `userdel` refuses. `userdel -f -r alice` forces it — but check first with `pgrep -u alice`.

---

## Why sudo > root login

1. **Attribution.** Every `sudo` command lands in `/var/log/auth.log` / journal with the real user's name. Direct root login is anonymous after the fact.
2. **Least privilege.** You spend 99% of your time as your normal user — typos can't destroy the system.
3. **Granular policy.** `alice` can run `apt update`; `bob` can restart nginx; only `you` can do everything.
4. **No shared password.** Three sysadmins each have their own password and key. Revoking access = removing one user, not rotating the root password and notifying everyone.
5. **Audit-friendly.** Compliance frameworks (PCI, SOC2) effectively require sudo + logging.

---

## Common pitfalls

- **`usermod -G` without `-a`** → loses group memberships silently.
- **Editing `/etc/sudoers` with vim/nano** instead of `visudo` → broken syntax locks you out.
- **`NOPASSWD: ALL`** for a regular user → equivalent to giving them root.
- **Reusing UIDs after deletion** → files owned by the *old* UID now belong to the *new* user. Either zero out leftover files or assign new UIDs above the old ones.
- **`su` vs `su -`** → without the dash you keep the calling user's env, including `PATH` — which can be a security issue.

→ [Module 09](../../module-09-networking/README.md)
