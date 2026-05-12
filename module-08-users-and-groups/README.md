# Module 08 — Users and Groups

**Phase:** System administration · **Time:** ~2 weeks · **Prereq:** Module 07

---

## 🗂️ The three account files — dissected

```
/etc/passwd     (world-readable, no passwords here)
─────────────────────────────────────────────────────
alice : x : 1001 : 1001 : Alice Doe : /home/alice : /bin/bash
  │     │    │      │        │            │             │
  user  pwd  UID   GID     GECOS        home         login shell
        ↑ "x" = real hash lives in /etc/shadow

/etc/shadow     (root-only)
─────────────────────────────────────────────────────
alice : $6$saltsalt$hashhashhash... : 19876 : 0 : 99999 : 7 : : :
         │                            │             │
       password hash ($6=SHA-512)   last change  max age days

/etc/group
─────────────────────────────────────────────────────
sudo : x : 27 : alice,bob
                 └─── members
```

## 🔐 sudo decision flow

```
  alice runs: sudo apt update
         │
         ▼
  ┌──────────────────────────────┐
  │ is alice in /etc/sudoers?    │── no ──> ❌ logged, denied
  └──────────────────────────────┘
         │ yes
         ▼
  ┌──────────────────────────────┐
  │ rule allows this command?    │── no ──> ❌ logged, denied
  └──────────────────────────────┘
         │ yes
         ▼
  ┌──────────────────────────────┐
  │ NOPASSWD set?                │── yes ──┐
  └──────────────────────────────┘         │
         │ no                               │
         ▼                                  │
  prompt password ── wrong ──> ❌ denied    │
         │ correct                          │
         ▼                                  │
   run as root <─────────────────────────── ┘
         │
         ▼
   ✅ logged in journal
```

---

## What you'll learn

- `/etc/passwd`, `/etc/shadow`, `/etc/group` — what's in them and why
- Creating, modifying, deleting users (`useradd`, `usermod`, `userdel`)
- Groups and group membership (`groupadd`, `gpasswd`)
- `sudo` and `/etc/sudoers` — the right way to grant privilege
- Password policies and account aging

## Readings

| Priority | Book | Chapter |
|---|---|---|
| Required | **ULSAH** | Ch. 8 — User Management |
| Recommended | **HLW** | Ch. 7 — System Configuration: Logging, System Time, Batch Jobs, and Users |
| Reference | **LCLSB** | Ch. 7 — File Permissions (revisit) |

## Key concepts

1. **Users are numbers (UIDs).** Names are just labels for them.
2. **UID 0 is root.** Anything running with UID 0 has full power. Be careful.
3. **`/etc/passwd` is world-readable** (no real passwords there!). `/etc/shadow` is not.
4. **`sudo` is preferred over `su -`** for almost everything. Always.
5. **Groups are for sharing resources.** A user can be in many groups.

## Exercises

In `exercises/`:
- Create a new user, set a password, log in as them
- Add the user to a group, verify with `id` and `groups`
- Configure sudoers to allow a user to run *only* `apt update` without a password
- Examine `/etc/shadow` (with sudo) and identify the hash algorithm used
- Lock and unlock an account

## Done when...

- You can create a user with the right home, shell, and groups in one command
- You understand the difference between primary and supplementary groups
- You'd never put `NOPASSWD: ALL` in sudoers for a regular user

→ [Module 09](../module-09-networking/README.md)

---

**Navigate:** [← Previous module](../module-07-text-processing/README.md) · [🏠 Home](../README.md) · [Next module →](../module-09-networking/README.md)
