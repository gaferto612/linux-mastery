# Module 10 — Storage and Filesystems

**Phase:** System administration · **Time:** ~2 weeks · **Prereq:** Module 09

## What you'll learn

- Disks, partitions, filesystems — three different things
- `df`, `du`, `lsblk`, `blkid`, `mount`, `/etc/fstab`
- Filesystem types: ext4, xfs, btrfs, tmpfs, swap
- Creating, mounting, and persisting a new disk
- LVM (Logical Volume Manager) basics
- Swap and when to use it

## Readings

| Priority | Book | Chapter |
|---|---|---|
| Required | **HLW** | Ch. 4 — Disks and Filesystems |
| Required | **ULSAH** | Ch. 20 — Storage |
| Recommended | **HLW** | Ch. 4 sections on LVM |

## Key concepts

1. **Disk → partition → filesystem → mount point.** Four layers.
2. **`df` is "disk free" (filesystem usage). `du` is "disk usage" (per file/dir).** Different numbers.
3. **`/etc/fstab` is what makes mounts persistent across reboots.** Edit carefully.
4. **A "filesystem" is a structure imposed on a partition.** ext4 ≠ xfs ≠ btrfs — different tradeoffs.
5. **Inodes can run out** even when disk space is free. `df -i` shows them.

## Exercises

In `exercises/`:
- Map your disk layout: physical disks → partitions → filesystems → mount points
- Add a virtual disk to your VM, partition it, format it, mount it, persist it in fstab
- Use `du` to find what's eating your disk space
- Create and use a tmpfs (RAM-backed filesystem)
- Add and remove swap

## Done when...

- You can walk someone through adding a new disk to a server
- You don't get scared by fstab
- You can answer "why is my disk full?" with confidence

→ [Module 11](../module-11-package-management/README.md)
