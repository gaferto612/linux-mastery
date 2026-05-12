# Module 11 — Package Management

**Phase:** System administration · **Time:** ~1 week · **Prereq:** Module 10

---

**In this module:** [📋 Exercises](exercises/README.md) · [✅ Solutions](solutions/README.md) · [📝 Notes](notes/my-notes.md)

## 📦 What "apt install nginx" actually does

```
   apt install nginx
        │
        ▼
   read /etc/apt/sources.list*
        │
        ▼
   fetch package index from repo
        │
        ▼
   resolve dependency graph
        │
        ▼
   download .deb files
   (cached in /var/cache/apt)
        │
        ▼
   verify signatures (GPG)
        │
        ▼
   run preinst scripts
        │
        ▼
   unpack files onto /
        │
        ▼
   run postinst scripts
   (e.g. enable service)
        │
        ▼
   ✅ nginx ready
```

## 🆚 The package-manager rosetta stone

```
                  Debian/Ubuntu     RHEL/Fedora      Arch
                  ─────────────     ───────────      ────
   refresh index  apt update        dnf check-update pacman -Sy
   install        apt install X     dnf install X    pacman -S X
   remove         apt remove X      dnf remove X     pacman -R X
   upgrade all    apt upgrade       dnf upgrade      pacman -Syu
   search         apt search X      dnf search X     pacman -Ss X
   owns this file dpkg -S /path     rpm -qf /path    pacman -Qo /path
   list files     dpkg -L pkg       rpm -ql pkg      pacman -Ql pkg
```

---

## What you'll learn

- How package managers actually work: repos, packages, dependencies
- `apt` (Debian/Ubuntu) — install, remove, upgrade, search
- `dpkg` — the low-level tool behind apt
- (Bonus) `dnf`/`yum` (RHEL family), `pacman` (Arch), `flatpak`, `snap`
- Building from source — and when *not* to

## Readings

| Priority | Book | Chapter |
|---|---|---|
| Required | **ULSAH** | Ch. 6 — Software Installation and Management |
| Recommended | **HLW** | Ch. 7 sections on package management |

## Key concepts

1. **A package is metadata + files + scripts.** The package manager installs, tracks, updates.
2. **The repo is just a website with files and an index.** Nothing magical.
3. **Dependencies are a graph.** Package managers solve them. Sometimes they fail (the dreaded "dependency hell").
4. **Always `update` before `install`.** The package list is cached.
5. **Avoid mixing package managers** (e.g., apt + pip system-wide). Use virtualenvs.

## Exercises

In `exercises/`:
- Search for, install, and remove a package
- See what a package contains and what owns a given file
- View installed packages, sort by size
- Add a third-party repo (safely)
- Pin a package version
- Clean up: cache, orphans, old kernels

## Done when...

- You can fully maintain a Debian/Ubuntu system from the terminal
- You know how to find which package provides `/usr/bin/whatever`
- You understand why "curl | sudo bash" is a bad idea

→ [Module 12](../module-12-services-and-systemd/README.md)

---

**Navigate:** [← Previous module](../module-10-storage-and-filesystems/README.md) · [🏠 Home](../README.md) · [Next module →](../module-12-services-and-systemd/README.md)
