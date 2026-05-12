# Module 06 — Solutions

## Exercise 1 — Make your scripts safe

Add to every script:

```bash
#!/bin/bash
set -euo pipefail
IFS=$'\n\t'        # bonus: stop word-splitting on spaces, only on newlines/tabs
```

What each flag catches in practice:

- `-e` → `cmd1; cmd2; cmd3` no longer silently continues when `cmd1` fails.
- `-u` → typo a variable name (`$fooo` instead of `$foo`) and the script aborts instead of expanding to empty string.
- `-o pipefail` → `false | true` was exit 0 (only the last status counted); now it's 1.

**Common breakage when you add `-e`:** code paths like `grep foo file` where "no match" is an *expected* exit 1. Two fixes:

```bash
grep foo file || true                       # accept non-zero
if grep -q foo file; then echo found; fi    # check explicitly
```

---

## Exercise 2 — getopts

```bash
#!/bin/bash
set -euo pipefail

verbose=0
out_dir="."
src_dir=""

while getopts ":d:o:v" opt; do
  case "$opt" in
    d) src_dir="$OPTARG" ;;
    o) out_dir="$OPTARG" ;;
    v) verbose=1 ;;
    \?) echo "Unknown option: -$OPTARG" >&2; exit 2 ;;
    :)  echo "Option -$OPTARG needs a value" >&2; exit 2 ;;
  esac
done

[[ -z "$src_dir" ]] && { echo "Usage: $0 -d <dir> [-o <out>] [-v]" >&2; exit 2; }
[[ -d "$src_dir" ]] || { echo "Not a directory: $src_dir" >&2; exit 1; }

stamp=$(date +%Y-%m-%d-%H-%M)
out="$out_dir/$(basename "$src_dir")-$stamp.tar.gz"
v_flag=$( [[ $verbose -eq 1 ]] && echo "v" || echo "" )

tar -c${v_flag}zf "$out" "$src_dir"
echo "Wrote $out"
```

Key points:

- The leading `:` in `":d:o:v"` enables *silent* error mode → you get to handle `\?` and `:` yourself.
- `d:` means `-d` takes an argument; `v` (no colon) doesn't.
- `OPTARG` holds the value of the current option.
- After the loop, `$1` is the *next* positional argument (use `shift $((OPTIND-1))` if you need them).

---

## Exercise 3 — Function library

`lib/log.sh`:

```bash
#!/bin/bash
# shellcheck disable=SC2034
LOG_COLOR=$( [[ -t 2 ]] && echo 1 || echo 0 )

_log() {
  local level="$1"; shift
  local color="$1"; shift
  local ts="$(date +'%Y-%m-%dT%H:%M:%S')"
  if [[ "$LOG_COLOR" -eq 1 ]]; then
    printf '\033[%sm[%s]\033[0m %s %s\n' "$color" "$level" "$ts" "$*" >&2
  else
    printf '[%s] %s %s\n' "$level" "$ts" "$*" >&2
  fi
}

log_info()  { _log "INFO"  "32" "$@"; }   # green
log_warn()  { _log "WARN"  "33" "$@"; }   # yellow
log_error() { _log "ERROR" "31" "$@"; }   # red
```

Use it:

```bash
#!/bin/bash
source "$(dirname "$0")/lib/log.sh"
log_info "starting"
log_warn "disk is at 85%"
log_error "could not connect"
```

`[[ -t 2 ]]` checks if stderr is a terminal — pipe it to a file and color codes vanish automatically.

---

## Exercise 4 — trap

```bash
#!/bin/bash
set -euo pipefail
tmpfile=$(mktemp)
trap 'rm -f "$tmpfile"' EXIT

echo "working with $tmpfile"
echo "data" > "$tmpfile"
# even if the next line dies, EXIT trap still runs
some_command_that_might_fail
```

`trap … EXIT` runs **on every exit path** — normal, `set -e` abort, `exit 5`, Ctrl-C (because Ctrl-C exits). That's why it's the right place for cleanup.

Multiple signals:

```bash
trap 'cleanup' EXIT INT TERM
```

---

## Exercise 5 — Arrays

```bash
#!/bin/bash
set -euo pipefail
files=(a.txt b.txt "name with spaces.txt")

for f in "${files[@]}"; do      # quoted [@] = each element is one word
  echo "→ $f"
done

# dynamic from directory
mapfile -t entries < <(find . -maxdepth 1 -type f -print0 | xargs -0 -n1 echo)
echo "found ${#entries[@]} files"
```

Critical quoting differences:

```bash
"${array[@]}"   ← N separate words (the right answer 95% of the time)
"${array[*]}"   ← single string, joined by IFS[0] (usually space)
${array[@]}     ← N words, word-split & globbed (almost never what you want)
```

---

## Exercise 6 — Parameter expansion

```bash
path="/home/user/photos/vacation.jpg"

"${path##*/}"   → vacation.jpg                  # strip longest prefix matching */
"${path#*/}"    → home/user/photos/vacation.jpg # strip shortest prefix
"${path%.*}"    → /home/user/photos/vacation    # strip shortest suffix matching .*
"${path%%.*}"   → /home/user/photos/vacation    # (same here; matters when multiple dots)
"${path%/*}"    → /home/user/photos             # strip last /thing  → dirname
"${path/photos/PHOTOS}"  → /home/user/PHOTOS/vacation.jpg  # first match
"${path//o/0}"  → /h0me/user/ph0t0s/vacati0n.jpg          # all matches
"${path:-default}"   → use "default" if path empty/unset
"${#path}"      → 33                              # length
```

Mnemonic: `#` removes from the front (top of the keyboard), `%` removes from the end. Doubled = greedy.

---

## Exercise 7 — find-duplicates.sh

```bash
#!/bin/bash
set -euo pipefail
[[ $# -eq 1 ]] || { echo "Usage: $0 <dir>" >&2; exit 2; }

find "$1" -type f -exec sha256sum {} + |
  sort |
  awk '
    {
      hash=$1; $1=""; file=substr($0,2)
      list[hash] = list[hash] file "\n"
      count[hash]++
    }
    END {
      for (h in count) if (count[h] > 1) {
        printf "── %s (%d copies)\n%s\n", h, count[h], list[h]
      }
    }
  '
```

Why `-exec … +` (not `\;`) — `+` batches many files into one `sha256sum` invocation; way faster. Why `sha256sum`, not `md5sum` — modern, no collisions in practice.

---

## Common pitfalls

- **`set -e` doesn't fire inside `if`, `&&`, `||`.** A failing command in `if cmd; then …` does NOT abort the script — that's the whole point of `if`. This is correct, but confuses people.
- **`local x=$(cmd)`** swallows the exit code of `cmd`, because `local` itself is a command that succeeds. Split into two lines if you depend on the exit status.
- **Forgetting to quote `"${arr[@]}"`** — silently breaks on filenames with spaces. Run with `set -x` and you'll see the arguments.
- **trap fires for ERR vs EXIT.** `trap … ERR` runs only on error; `trap … EXIT` runs every time. For cleanup, EXIT is what you want.

→ [Module 07](../../module-07-text-processing/README.md)
