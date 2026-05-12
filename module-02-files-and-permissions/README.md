# Module 02 — Files and Permissions

**Phase:** Foundations · **Time:** ~2 weeks · **Prereq:** Module 01

---

## 🔐 The permission infographic

```
                user   group  other
            ┌───────┬───────┬───────┐
  rwxr-x--- │  rwx  │  r-x  │  ---  │
            └───┬───┴───┬───┴───┬───┘
                │       │       │
                7       5       0      ←  chmod 750

      r = 4    w = 2    x = 1     (add them up)
```

| Symbol | Meaning on file  | Meaning on directory          |
|--------|------------------|-------------------------------|
| `r`    | read contents    | list entries                  |
| `w`    | modify contents  | create/delete entries         |
| `x`    | execute as program | enter the directory (`cd`)  |

## 🔗 Hard link vs symlink

```
Hard link — two names, one inode
    name: foo.txt ──┐
                    ├──> inode #42 (data blocks)
    name: bar.txt ──┘

Symlink — pointer to a name
    name: foo.txt  ──> inode #99 (data blocks)
                        ▲
                        │  (points to "foo.txt")
    name: link.txt  ────┘
```

> Delete `foo.txt`: hard-linked `bar.txt` still works. Symlinked `link.txt` becomes a dangling pointer.

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

---

**Navigate:** [← Previous module](../module-01-terminal-basics/README.md) · [🏠 Home](../README.md) · [Next module →](../module-03-processes-and-jobs/README.md)
