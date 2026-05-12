# Module 10 — Solutions

## Exercise 1 — Map your storage

```bash
lsblk                       # tree: disks → partitions → mounts
lsblk -f                    # adds FSTYPE, LABEL, UUID
df -h                       # mounted filesystems, sizes
df -i                       # INODE usage (run out of these and you can't create files)
mount | column -t           # full mount table
cat /etc/fstab              # what's mounted automatically at boot
```

Sample `lsblk -f`:

```
NAME   FSTYPE LABEL UUID                                 MOUNTPOINT
sda
├─sda1 vfat   EFI   1234-ABCD                            /boot/efi
├─sda2 ext4   boot  5d2b-…                               /boot
└─sda3 ext4   root  91f7-…                               /
```

The four layers: **disk** (`sda`) → **partition** (`sda3`) → **filesystem** (`ext4`) → **mount point** (`/`).

---

## Exercise 2 — Where's my disk going?

```bash
du -sh /var/*           # summary (-s), human-readable (-h), per top-level entry
du -sh ~/* | sort -h    # sort -h handles "1G", "200M" correctly
sudo du -h / 2>/dev/null | sort -rh | head -20
```

Friendlier alternative: `ncdu` — interactive, navigable.

> 🧠 **Why `df` and `du` disagree.** A file that's been deleted but is still open by a process still occupies disk space — `df` sees it (the filesystem hasn't reclaimed those blocks), `du` doesn't (no name to find). Restart the process holding it (or use `lsof | grep deleted` to find the culprit) and the space comes back.

---

## Exercise 3 — Add a new disk

```bash
lsblk                                # confirm new device, e.g. /dev/sdb (no partitions yet)

# Partition (interactive)
sudo fdisk /dev/sdb
   n       # new partition
   p       # primary
   <Enter> # default partition number
   <Enter> # default first sector
   <Enter> # default last sector (use whole disk)
   w       # write

# Format
sudo mkfs.ext4 /dev/sdb1

# Mount once
sudo mkdir /mnt/newdisk
sudo mount /dev/sdb1 /mnt/newdisk
df -h | grep newdisk

# Persist
sudo blkid /dev/sdb1        # copy the UUID
sudo cp /etc/fstab /etc/fstab.bak
echo "UUID=<paste>  /mnt/newdisk  ext4  defaults  0  2" | sudo tee -a /etc/fstab

# Verify WITHOUT rebooting
sudo umount /mnt/newdisk
sudo mount -a       # should re-mount; if it errors, FIX FSTAB before rebooting
```

The last two columns of fstab:

```
… defaults 0 2
           │ └── fsck pass: 0=skip, 1=root, 2=other fs
           └─── dump (legacy backup tool flag; almost always 0)
```

**Why UUID, not `/dev/sdb1`** — device names are not stable. Plug in a USB stick on next boot and your `sdb` may become `sdc`. UUID is set when the filesystem was created and survives forever.

---

## Exercise 4 — tmpfs

```bash
sudo mkdir /mnt/ramdisk
sudo mount -t tmpfs -o size=100M tmpfs /mnt/ramdisk
df -h | grep ramdisk             # tmpfs  100M …
echo "in ram" | sudo tee /mnt/ramdisk/note.txt
```

`/tmp` and `/dev/shm` are often tmpfs by default on modern distros — instant clear on reboot, no disk I/O. Look for them in `df -h`.

Use cases: caches that should disappear on reboot, scratch space for high-IOPS jobs, secrets you don't want hitting disk.

---

## Exercise 5 — Swap

```bash
swapon --show          # current swaps
free -h                # see Swap row

sudo fallocate -l 512M /swapfile
sudo chmod 600 /swapfile        # critical: world-readable swap = security hole
sudo mkswap /swapfile
sudo swapon /swapfile
swapon --show
```

Persist in fstab:

```
/swapfile none swap sw 0 0
```

Remove:

```bash
sudo swapoff /swapfile
sudo rm /swapfile
# also delete the fstab line
```

> Modern advice: keep some swap (1–4 GB) so the kernel can page out genuinely cold pages, but don't rely on it as "extra RAM". If your active set doesn't fit in RAM, add RAM — disk is 1000× slower.

---

## Why `df` says 90% but `du` shows 30 GB

Possible causes:

1. **Deleted but open files.** A logger crashed, the file was deleted, but the process still holds the fd. Restart the process or `lsof +L1` to find them.
2. **Hidden mounts.** Something is mounted on top of a directory you're `du`-ing — `du` walks the upper filesystem only.
3. **Reserved space.** ext4 reserves 5% by default for root. `tune2fs -m 1 /dev/sda1` lowers it to 1%.
4. **Different units / sparse files.** `du --apparent-size` vs default — different concepts of "size".
5. **Inode exhaustion.** Lots of tiny files. `df -i` shows it. Even with disk free, you can't create new files.

---

## LVM (when flexibility matters)

```
Physical disks  →  PV    pvcreate /dev/sdb /dev/sdc
                   │
                   ▼
              Volume Group   vgcreate data_vg /dev/sdb /dev/sdc
                   │
                   ▼
            Logical Volumes  lvcreate -L 50G -n web data_vg
                                       │
                                       ▼
                                  mkfs.ext4 /dev/data_vg/web
```

Win: resize logical volumes without repartitioning, add more PVs to extend a VG, snapshots.

Skip LVM if you don't need any of that — it's another moving part.

---

## Common pitfalls

- **Editing fstab and rebooting before testing.** Always `sudo mount -a` first — if it errors, fix in place. Otherwise next boot drops you to emergency mode.
- **Wrong fsck pass.** Setting fs-pass to 1 for non-root filesystems makes boot stupidly slow with mandatory fscks.
- **Mounting on a non-empty directory.** Mount hides whatever was there. You don't lose it — it's just inaccessible until you unmount.
- **Filling /var/log.** A runaway log can fill the root filesystem and prevent the system from booting cleanly. logrotate (Module 13) is the cure.
- **Using `fdisk` on GPT disks.** Older `fdisk` only handled MBR. Modern util-linux handles both, but `parted` / `gdisk` are explicit about GPT.

→ [Module 11](../../module-11-package-management/README.md)
