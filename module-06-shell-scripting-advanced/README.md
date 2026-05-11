# Module 06 — Shell Scripting Advanced

**Phase:** Bash scripting · **Time:** ~2 weeks · **Prereq:** Module 05

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
