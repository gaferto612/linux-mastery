# Module 07 — Text Processing (grep, sed, awk, and friends)

**Phase:** Bash scripting · **Time:** ~3 weeks · **Prereq:** Module 06

---

## 🧪 Anatomy of THE pipeline

```mermaid
flowchart LR
    log["access.log"] --> grep["grep 404"]
    grep --> cut["awk '{print $1}'<br/>extract IP"]
    cut --> sort["sort"]
    sort --> uniq["uniq -c<br/>count dupes"]
    uniq --> sort2["sort -rn<br/>desc by count"]
    sort2 --> head["head -10"]
    head --> out([Top 10 IPs<br/>causing 404s])
```

> ☝️ This 6-stage one-liner is the Unix philosophy in 80 columns. Memorize the **shape**, not the commands.

## 🛠️ Tool taxonomy — which one when?

```
┌────────┬──────────────────────────────────────────────────┐
│ grep   │ "does this line match?"   — filter by pattern    │
│ sed    │ "edit this line"          — substitute / delete  │
│ awk    │ "this line is a record"   — fields & arithmetic  │
│ cut    │ "give me column N"        — fast, fixed columns  │
│ sort   │ "order them"              — lexical or numeric   │
│ uniq   │ "fold duplicates"         — needs sorted input!  │
│ tr     │ "swap characters"         — translation table    │
│ xargs  │ "feed args from stdin"    — turn output into args│
│ tee    │ "Y-split the stream"      — to file AND stdout   │
└────────┴──────────────────────────────────────────────────┘
```

## 🔣 Regex — the minimum survival kit

```
 .        any single char        \.    literal dot
 *        0 or more of prev      +     1 or more (extended)
 ?        0 or 1 of prev (ext.)
 ^foo     starts with foo        foo$  ends with foo
 [abc]    one of a,b,c           [^a]  NOT a
 [0-9]    digit                  \d    digit (PCRE)
 \b       word boundary
 (a|b)    a or b   (extended: grep -E)
 \1 \2    backreferences in sed/awk
```

---

## What you'll learn

- `grep` properly: extended regex, recursive, context flags
- `sed` for substitution and stream editing
- `awk` for field-based processing (this is the big one)
- `cut`, `sort`, `uniq`, `tr`, `wc`, `tee` — the small champions
- `xargs` for chaining and parallel processing
- Regular expressions (basic + extended)

## Readings

| Priority | Book | Chapter |
|---|---|---|
| Required | **LCLSB** | Ch. 19 — Regular Expressions |
| Required | **LCLSB** | Ch. 20 — Sed Editor Basics |
| Required | **LCLSB** | Ch. 21 — The Awk Programming Language |
| Recommended | **LCLSB** | Ch. 22 — More with sed and gawk |
| Reference | **HLW** | Ch. 2 — Section on text processing |

## Key concepts

1. **Unix philosophy**: each tool does one thing well. You compose them with pipes.
2. **Regex is a language.** Worth investing a week to actually learn it, not just memorize patterns.
3. **`awk` is a complete programming language** disguised as a one-liner tool. Don't be afraid of it.
4. **`sed` is line-oriented**, edits streams. Default action: print every line, modified if matched.
5. **The 4-character pipeline** `grep ... | sort | uniq -c | sort -rn` is the most useful incantation in Unix.

## Exercises

In `exercises/`:
- Parse `/etc/passwd` with `awk` to list users with bash as their shell
- Use `grep` with context flags to find errors in `/var/log/syslog`
- Build a one-liner that finds the 10 most common words in a text file
- Rename a batch of files with `sed` and `mv` (or `rename`)
- Use `xargs` to run a command on the output of `find`

## Done when...

- You can write a working `awk` one-liner without checking docs
- Regex doesn't make you panic
- You reach for `awk` instead of writing a 10-line bash loop

→ [Module 08](../module-08-users-and-groups/README.md)
