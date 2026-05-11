# Module 08 — Exercises

⚠️ **All of these should be done on your VM, not on a production system.**

## Exercise 1 — Anatomy of /etc/passwd

```bash
head -5 /etc/passwd
```

For each line, identify all 7 fields. What's in the password field, and why isn't it the real password?

## Exercise 2 — Create a user the proper way

```bash
sudo useradd -m -s /bin/bash alice
sudo passwd alice
```

- `-m` creates the home directory
- `-s` sets the shell

Switch to that user:

```bash
su - alice
whoami; pwd; id
exit
```

## Exercise 3 — Groups

```bash
sudo groupadd developers
sudo usermod -aG developers alice
groups alice
id alice
```

Note: `-aG` (append) is critical. Without `-a`, `usermod -G` *replaces* groups.

## Exercise 4 — Look at /etc/shadow

```bash
sudo cat /etc/shadow | head
```

The second field has a structure like `$6$salt$hash`. The `$6` means SHA-512. Other values: `$1` MD5, `$2y` bcrypt, `$y` yescrypt.

## Exercise 5 — sudoers, carefully

**Always edit with `sudo visudo`** — it checks syntax before saving. A broken sudoers file can lock you out.

Add a line to give alice passwordless access to update packages only:

```
alice ALL=(ALL) NOPASSWD: /usr/bin/apt update
```

Switch to alice, try:

```bash
sudo apt update          # should work without password
sudo apt install vim     # should fail (or ask password depending on config)
```

## Exercise 6 — Lock and unlock

```bash
sudo passwd -l alice    # lock
sudo passwd -S alice    # check status
sudo passwd -u alice    # unlock
```

What does locking actually do? (Hint: look at the second field of alice's shadow entry before and after.)

## Exercise 7 — Cleanup

```bash
sudo userdel -r alice    # -r removes home directory too
sudo groupdel developers
```

## Reflection

Why is it good practice to *not* log in as root directly, even though you can? What does sudo give you that root login doesn't?
