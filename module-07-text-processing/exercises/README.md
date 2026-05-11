# Module 07 — Exercises

## Exercise 1 — grep gymnastics

1. Find all lines in `/etc/passwd` that contain "bash".
2. Same, but only count them: `grep -c`
3. Same, but show 2 lines of context before and after each match: `grep -B 2 -A 2`
4. Find files in `/etc` that contain the string "network" — recursively, case-insensitive: `grep -ril network /etc`
5. Use extended regex (`-E`) to find lines in `/etc/passwd` whose UID is exactly 4 digits.

## Exercise 2 — sed substitutions

1. `echo "Hello World" | sed 's/World/Linux/'` — basic substitution
2. Replace all occurrences in a file: `sed 's/old/new/g' file.txt`
3. In-place edit (be careful): `sed -i 's/old/new/g' file.txt`
4. Delete blank lines from a file: `sed '/^$/d'`
5. Print only lines 10-20 of a file: `sed -n '10,20p'`

## Exercise 3 — awk fundamentals

Given `/etc/passwd` is colon-separated:

```
root:x:0:0:root:/root:/bin/bash
```

Fields: 1=username, 2=password, 3=UID, 4=GID, 5=info, 6=home, 7=shell.

1. Print just usernames: `awk -F: '{print $1}' /etc/passwd`
2. Print usernames whose shell is `/bin/bash`: `awk -F: '$7 == "/bin/bash" {print $1}' /etc/passwd`
3. Print username + home dir, separated by tab: `awk -F: '{print $1 "\t" $6}'`
4. Count users with UID >= 1000 (regular users):  `awk -F: '$3 >= 1000 {count++} END {print count}'`
5. Sum a column of numbers. Save this in a file:

```
apples 5
oranges 3
bananas 7
```

Sum the second column: `awk '{sum+=$2} END {print sum}'`

## Exercise 4 — The classic incantation

Given any text file, find the 10 most common words:

```bash
cat file.txt \
  | tr -s '[:space:]' '\n' \
  | tr -d '[:punct:]' \
  | tr '[:upper:]' '[:lower:]' \
  | sort \
  | uniq -c \
  | sort -rn \
  | head -10
```

Try it on the output of `ls /usr/bin/` or any log file. Understand each stage of the pipeline — go through it slowly.

## Exercise 5 — find + xargs

1. Find all `.log` files in `/var/log` and count their lines (sum):

```bash
find /var/log -name "*.log" 2>/dev/null | xargs wc -l
```

2. Find all `.txt` files in your home and search them for "TODO":

```bash
find ~ -name "*.txt" -print0 | xargs -0 grep -l "TODO"
```

Why `-print0` and `-0`? Filenames can contain spaces and newlines. Null-separated is safer.

## Exercise 6 — Batch rename

Make some files: `touch IMG_001.jpg IMG_002.jpg IMG_003.jpg`.

Rename them all to `photo_001.jpg`, etc:

```bash
for f in IMG_*.jpg; do
  mv "$f" "${f/IMG_/photo_}"
done
```

Now do it with `rename` (if installed) — much shorter.

## Exercise 7 — Log analysis project

If your system has `/var/log/auth.log` (Ubuntu) or `/var/log/secure` (RHEL):

```bash
sudo grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn | head
```

What does this do? Read it left to right.

## Reflection

Pick *one* of these tools (grep, sed, or awk) and read its full man page over the course of a week. You don't have to memorize — just expose yourself.
