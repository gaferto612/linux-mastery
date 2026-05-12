# File Permissions Cheatsheet

## Reading `ls -l` output

```
-rwxr-xr--  1 alice  developers  4096 May 12 10:23 script.sh
│└┬┘└┬┘└┬┘  │  │       │          │      │            └ name
│ │  │  │   │  │       │          │      └ modified time
│ │  │  │   │  │       │          └ size in bytes
│ │  │  │   │  │       └ group
│ │  │  │   │  └ owner
│ │  │  │   └ link count
│ │  │  └ others permissions (r--)
│ │  └ group permissions   (r-x)
│ └ owner permissions      (rwx)
└ file type
```

## File types (first character)
| Char | Type |
|---|---|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `c` | Character device |
| `b` | Block device |
| `p` | Named pipe (FIFO) |
| `s` | Socket |

## Permissions
| Perm | On file | On directory |
|---|---|---|
| `r` (4) | Read contents | List contents |
| `w` (2) | Modify contents | Create/delete files in it |
| `x` (1) | Execute as program | Enter (cd into) it |

## Octal notation

Add up the bits per role: r=4, w=2, x=1.

| Octal | Symbolic | Meaning |
|---|---|---|
| `400` | `r--------` | Owner read only |
| `600` | `rw-------` | Owner read/write |
| `644` | `rw-r--r--` | Owner rw, others read — typical for regular files |
| `700` | `rwx------` | Owner full, no one else |
| `755` | `rwxr-xr-x` | Owner full, others read/execute — typical for scripts and dirs |
| `777` | `rwxrwxrwx` | Everyone everything — almost always wrong |

## chmod

```bash
chmod 755 file              # set absolute
chmod u+x file              # add execute for user
chmod g-w file              # remove write for group
chmod o-rwx file            # remove all for others
chmod a+r file              # all (u+g+o) add read
chmod -R 755 dir            # recursive
```

Letters:
- `u` user (owner)
- `g` group
- `o` others
- `a` all

## chown / chgrp

```bash
sudo chown alice file               # change owner
sudo chown alice:developers file    # change both at once
sudo chgrp developers file          # change group only
sudo chown -R alice:alice ~/Documents
```

## Special bits

| Bit | Octal prefix | What it does |
|---|---|---|
| setuid | 4000 | Run as file owner instead of you (only for executables) |
| setgid | 2000 | Run as group owner; on dirs, new files inherit dir's group |
| sticky | 1000 | On dirs, only the file owner can delete (`/tmp` has this) |

Examples:
```bash
chmod 4755 myprog       # setuid + 755
chmod 2755 dir          # setgid dir
chmod 1777 dir          # like /tmp
```

`ls -l` shows these as `s` (in place of x for setuid/setgid) and `t` (in place of x in others column for sticky):
- `-rwsr-xr-x` = setuid + 755 (like `/usr/bin/passwd`)
- `drwxrwxrwt` = 1777 (like `/tmp`)

## Defaults: umask

```bash
umask              # show your umask, e.g. 0022
umask 0077         # restrict — new files become 600, new dirs 700
```

New file permissions = `0666 & ~umask`. With umask 022 → 644.
New dir permissions  = `0777 & ~umask`. With umask 022 → 755.

## find by permission

```bash
find / -perm -4000          # all setuid files (security audit!)
find . -perm 644            # exactly 644
find . -perm /u+w           # any file writable by owner
find . -perm -u+x,-g+x      # both owner AND group execute set
```

## Common gotchas

- `chmod 777` is almost never the right fix. Find the actual permission problem.
- Permissions on a symlink are usually ignored — the target's matter.
- A user can read a file only if they have `x` on every directory in its path.
- `rm` doesn't need write permission *on the file* — it needs write+execute on the *parent directory*.

---

[← Back to course home](../README.md)
