# Module 11 — Exercises

(Assumes Debian/Ubuntu. If you're on RHEL/Fedora, mentally swap `apt` → `dnf`, `dpkg` → `rpm`.)

## Exercise 1 — Basics

```bash
sudo apt update                  # refresh package lists
apt search htop                  # search
apt show htop                    # info
sudo apt install htop
sudo apt remove htop             # uninstall, keep config
sudo apt purge htop              # uninstall + config
sudo apt autoremove              # remove orphan deps
```

## Exercise 2 — Upgrading

```bash
sudo apt update
sudo apt upgrade                 # upgrade installed packages
sudo apt full-upgrade            # may also install/remove to satisfy deps
```

(Don't run `dist-upgrade` casually — it's an OS-version upgrade on Debian.)

## Exercise 3 — Inspect packages

```bash
dpkg -l                          # all installed packages
dpkg -l | wc -l                  # how many?
dpkg -L coreutils                # files installed by coreutils
dpkg -S /usr/bin/ls              # which package owns this file?
apt-file search filename         # find a file across packages (install apt-file first)
```

In your notes: which package owns `/usr/bin/grep`? `/etc/passwd`?

## Exercise 4 — Sort by size

```bash
dpkg-query -Wf '${Installed-Size}\t${Package}\n' | sort -rn | head -20
```

What are your fattest packages?

## Exercise 5 — Adding a third-party repo (carefully)

The "right" way for, e.g., Docker:

```bash
# 1. Add the signing key (so package signatures can be verified)
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 2. Add the repo with that key
echo "deb [signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list

# 3. Update and install
sudo apt update
```

Read each line. Don't just paste blindly. (This is why "curl | sudo bash" is dangerous — you have no idea what's happening.)

## Exercise 6 — Pin a version

```bash
apt-cache policy htop            # see candidate version
sudo apt install htop=3.0.5-7    # specific version
sudo apt-mark hold htop          # prevent upgrades
sudo apt-mark unhold htop
```

## Exercise 7 — Clean up

```bash
sudo apt autoremove              # orphan dependencies
sudo apt autoclean               # cached .deb files no longer in repo
sudo apt clean                   # all cached .debs
sudo apt list --installed | head
```

## Exercise 8 — Build from source (once)

Pick a small tool from GitHub with a `Makefile`. Clone, read the README, `make`, `sudo make install`. Then:

- Where did it install?
- Can you `apt remove` it? (No.)
- How would you uninstall it? (Usually `sudo make uninstall` — if the makefile defines it.)

Lesson: package managers track files for you. Source installs don't.

## Reflection

Why is mixing system Python packages (`sudo pip install`) with apt-installed Python packages a problem? (Hint: who tracks what?)
