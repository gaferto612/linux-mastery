# Module 02 — Files and Permissions

**Phase:** Foundations · **Time:** ~2 weeks · **Prereq:** Module 01

---

## What you'll learn

- The Linux filesystem hierarchy: what lives where and why
- Symbolic links vs hard links — the real difference
- File permissions: read/write/execute, user/group/other
- `chmod`, `chown`, `chgrp` — changing ownership and access
- Why permissions exist (security, multi-user systems)

---

## Readings

| Priority | Book | Chapter |
|---|---|---|
| Required | **HLW** | Ch. 2 §2.11–2.20 (files, links, archives) |
| Required | **HLW** | Ch. 4 — Disks and Filesystems (skim sections on permissions) |
| Recommended | **LCLSB** | Ch. 7 — Understanding Linux File Permissions |
| Recommended | **ULSAH** | Ch. 5 — The Filesystem |

---

## Key concepts

1. **Everything is a file.** Even devices (`/dev/sda`), even running process info (`/proc`). Profound — sit with it.
2. **The FHS (Filesystem Hierarchy Standard)** is why `/etc`, `/var`, `/usr` look the same on every Linux distro.
3. **Three permission triplets:** owner, group, others. Each gets r/w/x. That's it.
4. **Octal notation:** `chmod 755` = `rwxr-xr-x`. Learn to read both.
5. **Hard links share inodes; symlinks are pointers.** Deleting the original breaks a symlink but not a hard link.

---

## Exercises (in `exercises/`)

You'll:
- Map the `/` directory and write down what each top-level folder is for
- Create files with different permission combinations
- Set up a "private" file only you can read
- Create both a hard link and a symlink to the same file, then delete the original — observe what happens
- Use `find` to locate files by permission

---

## Done when...

- You can read `-rwxr-x---` at a glance and know who can do what
- You understand why `chmod 777` is almost always wrong
- You can explain a symlink to a friend

→ [Module 03](../module-03-processes-and-jobs/README.md)
