# Module 02 — Solutions

## Exercise 1 — FHS

Quick reference:
- `/etc` — system-wide configuration
- `/var` — variable data: logs, mail, databases, caches
- `/usr` — installed software (most binaries and libraries live here)
- `/home` — user home directories
- `/tmp` — temporary files (often wiped on reboot)
- `/opt` — optional third-party software
- `/proc` — virtual filesystem; live kernel/process info
- `/dev` — device files (disks, terminals, etc.)
- `/boot` — kernel and bootloader files
- `/root` — root user's home directory (different from `/`!)

`man hier` is the canonical reference.

## Exercise 2 — Permission strings

```
-rwxr-xr--   regular file, owner: rwx, group: r-x, others: r--
-rw-------   regular file, owner only can read/write
drwxrwxr-x   directory, owner+group can do everything, others can enter & list
lrwxrwxrwx   symbolic link (permissions on a symlink itself are usually ignored — the target's permissions are what matter)
-rwsr-xr-x   the 's' is setuid — when executed, the program runs with the owner's privileges instead of yours. Classic example: /usr/bin/passwd
```

First character: `-` regular file, `d` directory, `l` symlink, `c` character device, `b` block device.

## Exercise 3 — chmod

```bash
touch secret.txt
chmod 600 secret.txt        # only owner can read/write
chmod u+x secret.txt        # add execute for user
chmod o-r secret.txt        # remove read for others (already gone, but harmless)
ls -l secret.txt
```

`600` = `rw-------`. The three octal digits are owner/group/others, where r=4, w=2, x=1 (add them).

## Exercise 4 — Ownership

```bash
touch myfile.txt
chown root myfile.txt       # → "Operation not permitted"
sudo chown root myfile.txt  # works
sudo chown $USER myfile.txt # change it back
```

**Why?** Regular users can't give files away or take other users' files. Only root can change ownership. This prevents shenanigans like "make this file belong to root so it has more privileges."

## Exercise 5 — Links

```bash
echo "hello" > original.txt
ln original.txt hard.txt
ln -s original.txt soft.txt
ls -li
```

`original.txt` and `hard.txt` share the same inode number — they are literally the same file with two names.
`soft.txt` is its own tiny file that contains the path "original.txt".

```bash
rm original.txt
cat hard.txt    # "hello" — still works! The data lives at the inode, and hard.txt still points to it.
cat soft.txt    # error — the path "original.txt" no longer exists, and soft.txt is a dangling symlink.
```

**This is the single best way to understand inodes.** Files don't have names; *directories* contain name → inode mappings.

## Exercise 6 — find

```bash
find ~ -type f -executable          # 1
find /etc -type f -size +100k       # 2
find /tmp -type f -mtime -1         # 3 (modified < 1 day ago)
```

`find` is a *huge* command. Learn one flag at a time.

## Exercise 7 — Locked out

`chmod 000` removes all permissions, including for you. You can't enter the directory or read anything in it. You can still see it from outside (because the *parent* directory's permissions decide whether you can list children's names).

You fix it with `chmod` because the permission check happens against the file's permissions, *but* the kernel always lets the owner change their own file's permissions back. This is a key safety rail.
