# Module 02 — Exercises

## Exercise 1 — Mapping the filesystem

`ls /` and for each top-level directory, find out what it's for. Use `man hier` for help. Write 1-line descriptions in your notes for: `/etc`, `/var`, `/usr`, `/home`, `/tmp`, `/opt`, `/proc`, `/dev`, `/boot`, `/root`.

## Exercise 2 — Reading permission strings

For each line below, write out who can do what:

```
-rwxr-xr--
-rw-------
drwxrwxr-x
lrwxrwxrwx
-rwsr-xr-x
```

What does the very first character mean? What is `s` doing in the last one? (Bonus: look up "setuid".)

## Exercise 3 — chmod with numbers and letters

1. Create a file `secret.txt`.
2. Make it readable only by you. (Hint: `600`)
3. Now use `chmod` with letters (e.g., `chmod u+x`, `chmod o-r`) to:
   - Add execute permission for the owner
   - Remove read permission for "others" if it has any
4. Verify with `ls -l` after each step.

## Exercise 4 — Ownership

1. Create a file as your normal user.
2. Try to change its ownership to root with `chown`. What happens? Why?
3. Now do it with `sudo`. What's different?
4. Change ownership back to yourself.

## Exercise 5 — Links

1. Create a file `original.txt` with some content (`echo "hello" > original.txt`).
2. Create a hard link to it: `ln original.txt hard.txt`
3. Create a symbolic link to it: `ln -s original.txt soft.txt`
4. `ls -li` — note the inode numbers. Which two files share an inode?
5. Delete `original.txt`. Now try to `cat hard.txt` and `cat soft.txt`. What happens with each? Why?

## Exercise 6 — Find by permission

1. Use `find` to list all files in your home directory that are executable by you.
2. Use `find` to list all files in `/etc` larger than 100 KB.
3. Use `find` to find files in `/tmp` modified in the last day.

(Don't worry about mastering `find` now — you'll come back to it.)

## Exercise 7 — The dangerous one

Make a directory `danger/` and put a few files in it.

Run `chmod -R 000 danger/`. Try to `cd` into it. What happens?

Now fix it: `chmod -R 755 danger/`.

**Lesson:** permissions can lock *you* out. With great power...
