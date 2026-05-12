# Module 06 — Shell Scripting Advanced

**Phase:** Bash scripting · **Time:** ~2 weeks · **Prereq:** Module 05

---

**In this module:** [📋 Exercises](exercises/README.md) · [✅ Solutions](solutions/README.md) · [📝 Notes](notes/my-notes.md)

## 🛡️ The "safe bash" headers — what each flag does

```
#!/bin/bash
set -e          ┐
set -u          ├─ or simply:  set -euo pipefail
set -o pipefail ┘
```

```
  -e   exit on any error           (no more silently-failing scripts)
  -u   error on unset variables    (catches typos in $foo vs $fooo)
  -o pipefail
       pipeline fails if ANY stage fails
       (otherwise only the last command's status counts)
```

## ✨ Parameter expansion mini-cheatsheet

```
  ${var}              value of var
  ${var:-default}     use default if var unset/empty
  ${var:=default}     same, and ASSIGN default
  ${var:?err msg}     error out with message if unset
  ${#var}             length
  ${var%.txt}         strip suffix    ".txt"
  ${var##*/}          strip prefix to last "/"  (basename)
  ${var//foo/bar}     replace ALL "foo" with "bar"
  ${var^^}            uppercase   ${var,,} lowercase
```

## 🪤 trap — guaranteed cleanup

```
  script starts
       │
       ▼
  mktemp tmpfile
       │
       ▼
  trap 'rm -f $tmpfile' EXIT
       │
       ▼
  do work...
       │
       ▼
  ┌──────────────┐
  │ exit reason? │── normal exit ──┐
  │              │── error ────────┤
  │              │── Ctrl-C / kill ┤
  └──────────────┘                 │
                                   ▼
                              cleanup runs
                                   │
                                   ▼
                            🟢 tmpfile gone
```

---

## What you'll learn

- Functions and return values
- Arrays (indexed and associative)
- String manipulation: parameter expansion (`${var%.txt}`, `${var//foo/bar}`)
- `getopts` for proper command-line option parsing
- Trap handlers for cleanup on exit
- Debugging: `set -e`, `set -x`, `set -u`, `set -o pipefail`
- Writing scripts that are *safe* by default

## Readings

| Priority | Book | Chapter |
|---|---|---|
| Required | **LCLSB** | Ch. 16 — Script Control |
| Required | **LCLSB** | Ch. 17 — Creating Functions |
| Recommended | **LCLSB** | Ch. 18 — Writing Scripts for Graphical Desktops |
| Reference | **LCLSB** | Ch. 24 — Using Common Bash Commands |

## Key concepts

1. **`set -euo pipefail` at the top of every serious script.** It turns silent failures loud.
2. **Functions in bash don't really have parameters** — they receive `$1`, `$2`, etc. just like scripts.
3. **Parameter expansion is bash's hidden gem.** `${var:-default}`, `${var%.ext}`, `${var//pat/rep}`. Worth memorizing.
4. **`trap` lets you run code on exit/signals.** Use it for cleanup of temp files.
5. **Arrays exist** but are stranger than in other languages. Quote them: `"${array[@]}"`.

## Exercises

In `exercises/`:
- Convert your Module 5 scripts to use `set -euo pipefail`
- Write a script with proper `getopts` argument parsing (`-v`, `-o output.txt`, etc.)
- Build a function library: `lib/log.sh` with `log_info`, `log_warn`, `log_error`
- Use `trap` to delete a temp file even if the script is killed
- Process a list of files using an array

## Done when...

- Every script you write starts with `#!/bin/bash` + `set -euo pipefail`
- You reach for functions instead of copy-pasting
- shellcheck gives you few warnings

→ [Module 07](../module-07-text-processing/README.md)

---

**Navigate:** [← Previous module](../module-05-shell-scripting-basics/README.md) · [🏠 Home](../README.md) · [Next module →](../module-07-text-processing/README.md)
