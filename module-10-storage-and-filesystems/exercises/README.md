# Module 10 — Exercises

## Exercise 1 — Map your storage

```bash
lsblk                   # tree of block devices
lsblk -f                # with filesystem info
df -h                   # mounted filesystems, human readable
df -i                   # inode usage
mount | column -t       # what's mounted where
cat /etc/fstab          # persistent mounts
```

Draw a diagram in your notes: disk → partitions → filesystems → mount points.

## Exercise 2 — Where's my disk going?

```bash
du -sh /var/*           # size of each subdir under /var
du -sh ~/* | sort -h    # your home, sorted
```

Find the biggest offenders. Useful one-liner:

```bash
sudo du -h / 2>/dev/null | sort -rh | head -20
```

(`ncdu` is a friendlier interactive version. Install it.)

## Exercise 3 — Add a new disk (VM only)

In your VM software, add a new virtual disk (e.g., 1 GB). Then in the VM:

```bash
lsblk                                # find the new device, e.g., /dev/sdb
sudo fdisk /dev/sdb                  # interactive: n, p, default, default, w
sudo mkfs.ext4 /dev/sdb1             # format
sudo mkdir /mnt/newdisk
sudo mount /dev/sdb1 /mnt/newdisk
df -h | grep newdisk
```

Now make it persistent. Get the UUID:

```bash
sudo blkid /dev/sdb1
```

Add to `/etc/fstab` (backup first!):

```
UUID=xxxxx  /mnt/newdisk  ext4  defaults  0  2
```

Test without reboot:

```bash
sudo umount /mnt/newdisk
sudo mount -a            # mount everything in fstab
df -h | grep newdisk     # should still be mounted
```

If `mount -a` errors out, fix fstab *before* rebooting — otherwise the system may fail to boot. (This is why we use `mount -a` to test.)

## Exercise 4 — tmpfs

```bash
sudo mkdir /mnt/ramdisk
sudo mount -t tmpfs -o size=100M tmpfs /mnt/ramdisk
df -h | grep ramdisk
echo "this lives in RAM" | sudo tee /mnt/ramdisk/note.txt
```

Reboot. The file is gone. tmpfs lives in RAM.

## Exercise 5 — Swap

Check current swap:

```bash
swapon --show
free -h
```

Create a 512 MB swap file:

```bash
sudo fallocate -l 512M /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
swapon --show
```

To make persistent: add to fstab: `/swapfile none swap sw 0 0`.

To remove: `sudo swapoff /swapfile && sudo rm /swapfile` (and remove fstab line).

## Reflection

What's the difference between `df` showing 90% used and `du` showing only 30 GB of files on a 100 GB disk? (Hint: deleted-but-open files.)
