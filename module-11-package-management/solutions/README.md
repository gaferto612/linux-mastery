# Module 11 — Solutions

## Exercise 1 — Basics

```bash
sudo apt update                # refresh package index from configured repos
apt search htop                # by name + short description
apt show htop                  # detailed metadata
sudo apt install htop
sudo apt remove htop           # uninstall but keep config files
sudo apt purge htop            # uninstall + delete config files
sudo apt autoremove            # remove orphaned dependencies
```

`apt update` does NOT install anything. It pulls down `Packages` / `Release` files from each repo into `/var/lib/apt/lists/` so apt knows what's available. Run it before any `install`/`upgrade`.

The `remove` vs `purge` distinction matters when you re-install: `remove` keeps `/etc/foo/foo.conf` and your settings come back; `purge` wipes them.

---

## Exercise 2 — Upgrading

```bash
sudo apt update && sudo apt upgrade
sudo apt full-upgrade          # may add/remove packages to resolve new deps
```

- `upgrade` — never removes installed packages. If an upgrade requires removing something, it's skipped.
- `full-upgrade` (formerly `dist-upgrade`) — allowed to remove packages to satisfy the dep graph. Use this when packages are being "kept back".
- **Major OS version upgrades** (Ubuntu 22.04 → 24.04) use `do-release-upgrade`, not apt.

---

## Exercise 3 — Inspect packages

```bash
dpkg -l                        # all installed (status line at start uses ii/rc codes)
dpkg -l | wc -l
dpkg -L coreutils              # all files this package installed
dpkg -S /usr/bin/ls            # → coreutils: /usr/bin/ls
apt-file search filename       # which package PROVIDES this file (even if not installed)
```

`/usr/bin/grep` → owned by `grep` package.
`/etc/passwd` → owned by `passwd` (or `base-passwd` on some systems — it's a generated/managed file).

dpkg status codes:

```
ii   installed
rc   removed but config files remain
hi   on hold (apt-mark hold)
un   never installed
```

---

## Exercise 4 — Sort by size

```bash
dpkg-query -Wf '${Installed-Size}\t${Package}\n' | sort -rn | head -20
```

`-W` query format, `${Installed-Size}` is in KiB. Common heavyweights: kernel images, `linux-firmware`, `texlive-*`, language packs, IDEs.

For "what just got installed and is big":

```bash
grep " install " /var/log/dpkg.log | tail -50
```

---

## Exercise 5 — Adding a third-party repo

```bash
# 1. Trust the signing key (modern, keyring-based)
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 2. Add the repo, pinned to that one key
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list

sudo apt update
```

Why this dance:

- `[signed-by=…]` ties this specific repo to that specific key. **Older** `apt-key add` made keys system-wide — any added repo could publish "Linux kernel" updates. Bad.
- Keyrings live in `/etc/apt/keyrings` (or `/usr/share/keyrings` for distro-shipped). Drop a key once, reference it from each list file.

`curl … | sudo bash` is the dangerous version because you have no chance to inspect what runs.

---

## Exercise 6 — Pin a version

```bash
apt-cache policy htop
# htop:
#   Installed: (none)
#   Candidate: 3.0.5-7
#   Version table:
#      3.0.5-7 500
#         500 http://archive.ubuntu.com/ubuntu jammy/main amd64 Packages

sudo apt install htop=3.0.5-7
sudo apt-mark hold htop          # exclude from upgrades
apt-mark showhold                # list held packages
sudo apt-mark unhold htop
```

For more sophisticated pinning (prefer a backports version, downgrade an entire repo's priority), see `/etc/apt/preferences.d/` and `man apt_preferences`.

---

## Exercise 7 — Clean up

```bash
sudo apt autoremove        # packages installed as deps that nothing needs now
sudo apt autoclean         # cached .debs for versions no longer in any repo
sudo apt clean             # ALL cached .debs (recoverable disk space)
```

`/var/cache/apt/archives/` is where the `.deb` files sit after download. Useful to keep on bandwidth-constrained machines, throw away on tiny VMs.

Old kernels accumulate in `/boot` — `sudo apt autoremove --purge` typically clears them, or list them with `dpkg -l 'linux-image-*'` and remove all but the current (`uname -r`) and one previous.

---

## Exercise 8 — Build from source

A typical session:

```bash
git clone https://github.com/example/tool.git
cd tool
make
sudo make install         # usually copies to /usr/local/{bin,lib,share}
```

- It usually goes to `/usr/local/...`. By convention `/usr/local` is for "things not managed by the distro package manager".
- `apt remove tool` does NOT work — apt has no record of it.
- Uninstall via `sudo make uninstall` IF the Makefile implements it. Many don't. You'd then `rm` each file by hand from `make install`'s output.

> **The lesson:** package managers track files. Source installs leak files into your system. Avoid mixing the same software through both channels.

For Python the equivalent issue is `sudo pip install` — installs into system Python, can fight with apt-managed python packages. Use `pipx`, virtualenvs, or `--user`.

---

## Common pitfalls

- **Skipping `apt update`** → installs use stale index, miss security fixes.
- **`apt-key add`** (deprecated) → trusts a key globally. Use `signed-by=` instead.
- **`curl | sudo bash` install scripts** → can't audit, runs as root.
- **`sudo pip install`** alongside apt → who owns `/usr/lib/python3/...`? Now nobody.
- **Holding security-critical packages** (openssh, openssl) → you'll miss CVE fixes. Hold sparingly.

→ [Module 12](../../module-12-services-and-systemd/README.md)
