# Module 05 — Solutions

## Exercise 1 — Your first script

```bash
#!/bin/bash
read -p "What's your name? " name
echo "Hello, $name!"
```

`read -p` prints a prompt and reads a line into `$name`. The shebang line tells the kernel which interpreter to use — without it, the kernel will try to execute the file as machine code and fail.

---

## Exercise 2 — Variables and quoting

```bash
greeting="hello world"
echo $greeting               # hello world      ← word-splits, then echo joins with one space
echo "$greeting"             # hello world      ← preserved as one argument
echo '$greeting'             # $greeting        ← single quotes = literal, no expansion
echo "$greeting     extra spaces"   # hello world     extra spaces  ← preserved
echo $greeting     extra spaces     # hello world extra spaces      ← collapsed
```

**Rule of thumb:** quote `"$var"` *every single time* unless you have a specific reason not to. Unquoted variables get split on whitespace and glob-expanded. This is the #1 source of script bugs.

---

## Exercise 3 — A backup script

```bash
#!/bin/bash
if [[ $# -ne 1 ]]; then
  echo "Usage: $0 <directory>" >&2
  exit 1
fi

dir="$1"
if [[ ! -d "$dir" ]]; then
  echo "Not a directory: $dir" >&2
  exit 1
fi

stamp=$(date +%Y-%m-%d-%H-%M)
out="$(basename "$dir")-$stamp.tar.gz"
tar -czf "$out" "$dir"
echo "Backup complete: $out"
```

Why `$(basename "$dir")` — so `./backup.sh /home/me/docs` produces `docs-…tar.gz`, not `/home/me/docs-…tar.gz`.

Why `>&2` — error messages go to stderr, not stdout, so they don't pollute pipelines.

---

## Exercise 4 — Conditional logic

```bash
#!/bin/bash
file="$1"
case "${file,,}" in           # ${var,,} = lowercase (bash 4+)
  *.jpg|*.jpeg|*.png|*.gif|*.webp) echo "Image" ;;
  *)                               echo "Not an image" ;;
esac
```

`case` is cleaner than nested `if [[ … ]]` when matching against multiple patterns. `;;` ends each branch. `*)` is the default.

---

## Exercise 5 — Loops

```bash
#!/bin/bash
shopt -s nullglob             # *.txt expands to nothing if no matches
for f in *.txt; do
  lines=$(wc -l < "$f")
  echo "$f: $lines"
done
```

`wc -l < file` (redirection) instead of `wc -l file` so wc outputs just the count, not "count filename".

`nullglob` prevents the classic bug where the loop runs once with `f="*.txt"` if no `.txt` files exist.

---

## Exercise 6 — Arguments and validation

```bash
#!/bin/bash
if [[ $# -lt 1 ]]; then
  echo "Usage: $0 name1 [name2] [name3]" >&2
  exit 1
fi
if [[ $# -gt 3 ]]; then
  echo "Too many names" >&2
  exit 2
fi
for name in "$@"; do
  echo "Hello, $name!"
done
```

`"$@"` (quoted) = each argument as a separate word, even if it contains spaces. `$@` unquoted = word-split. **Always use `"$@"`.**

---

## Exercise 7 — Exit codes

```bash
#!/bin/bash
f="$1"
if [[ ! -e "$f" ]]; then
  echo "Not found"
  exit 1
fi
if [[ ! -r "$f" ]]; then
  echo "Not readable"
  exit 2
fi
echo "OK"
exit 0
```

`-e` exists, `-r` readable, `-w` writable, `-x` executable, `-f` regular file, `-d` directory, `-L` symlink. Memorize a handful — these test operators are everywhere.

Check exit codes with `$?` immediately after running:

```bash
./check.sh /etc/hostname; echo "Exit: $?"   # Exit: 0
./check.sh /nope; echo "Exit: $?"           # Exit: 1
```

---

## Exercise 8 — The shellcheck habit

Typical shellcheck flags you'll see:

- **SC2086** — "Double quote to prevent globbing and word splitting." Fix: `"$var"`.
- **SC2046** — same idea for `$(cmd)`. Fix: `"$(cmd)"`.
- **SC2155** — "Declare and assign separately" (`local x="$(cmd)"` hides exit codes; split into `local x; x="$(cmd)"`).
- **SC2148** — missing shebang.

> Run shellcheck in your editor (it has plugins for vim/VS Code/etc.) so you see issues as you type.

---

## The pattern that emerges

Every script in this module follows the same shape:

```
shebang
   ↓
validate arguments   →   error + non-zero exit
   ↓
do the work          →   on failure: error + non-zero exit
   ↓
exit 0
```

Internalize this and you've already escaped 80% of "works on my machine" script bugs.

---

## Common pitfalls

- **Unquoted variables.** `rm $file` deletes anything if `$file` contains spaces or `*`.
- **`==` vs `=` in `[`.** POSIX `[` uses `=`. Bash `[[` accepts both. Stick with `=` in POSIX, `==` in `[[`.
- **`echo` weirdness.** Different `echo` implementations handle `-e` and backslashes differently. Use `printf` when you care about portability.
- **No shebang.** A script without `#!/bin/bash` runs in whatever shell the user invoked — may not be bash. Always include one.

→ [Module 06](../../module-06-shell-scripting-advanced/README.md)
