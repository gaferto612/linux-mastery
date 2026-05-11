# Module 06 — Exercises

## Exercise 1 — Make your scripts safe

Take every script from Module 5. Add to the top:

```bash
#!/bin/bash
set -euo pipefail
```

- `-e`: exit on error
- `-u`: error on unset variables
- `-o pipefail`: a pipeline fails if any command in it fails

Re-run them. Did anything break? Fix it.

## Exercise 2 — Proper option parsing with getopts

Rewrite `backup.sh` to support:

```
backup.sh -d <dir> -o <output-dir> [-v]
```

- `-d`: directory to back up (required)
- `-o`: where to put the archive (default: current dir)
- `-v`: verbose mode (use `tar -v`)

Use `getopts` (not `getopt`). Read `help getopts`.

## Exercise 3 — Function library

Create `lib/log.sh`:

```bash
log_info()  { echo "[INFO]  $*"; }
log_warn()  { echo "[WARN]  $*" >&2; }
log_error() { echo "[ERROR] $*" >&2; }
```

Source it in another script: `source lib/log.sh` then use the functions.

Now extend with:
- Timestamp prefix on each message
- Optional color output if stderr is a terminal (`[ -t 2 ]`)

## Exercise 4 — Cleanup with trap

Write a script that:
1. Creates a temp file: `tmpfile=$(mktemp)`
2. Sets a trap: `trap 'rm -f "$tmpfile"' EXIT`
3. Does some work with the temp file
4. Exits normally

Verify the temp file is gone. Now make the script `exit 1` partway through. Verify the temp file is *still* gone. That's the magic of `trap`.

## Exercise 5 — Arrays

Write a script that:
1. Defines an array of filenames: `files=(a.txt b.txt c.txt)`
2. Loops through them: `for f in "${files[@]}"; do ...`
3. Builds the array dynamically from a directory listing
4. Reports how many elements: `${#files[@]}`

Be careful with quoting.

## Exercise 6 — Parameter expansion practice

Given `path="/home/user/photos/vacation.jpg"`:

```bash
echo "${path##*/}"      # vacation.jpg  — strip everything before last /
echo "${path%.*}"       # /home/user/photos/vacation — strip extension
echo "${path%/*}"       # /home/user/photos — directory part
echo "${path/photos/PHOTOS}"   # replace first occurrence
echo "${path//o/0}"     # replace all 'o' with '0'
```

Try each. Get them in your fingers.

## Exercise 7 — A real tool

Build `find-duplicates.sh`:
1. Takes a directory as argument
2. Finds files with identical content (use `md5sum` or `sha256sum`)
3. Prints groups of duplicates

Hint: `find ... -exec md5sum {} \; | sort | awk ...`. You'll use Module 7 skills here too — come back to this exercise after Module 7 if you want.

## Reflection

Which `set` option saved you from a bug? Note it down. Next time you skip `set -euo pipefail`, you'll regret it.
