# Module 07 — Solutions

## Exercise 1 — grep gymnastics

```bash
grep bash /etc/passwd                       # 1
grep -c bash /etc/passwd                    # 2  count only
grep -B 2 -A 2 bash /etc/passwd             # 3  context (-C 2 is the shortcut for "both")
grep -ril network /etc 2>/dev/null          # 4  -r recursive, -i case-insensitive, -l names only
grep -E '^[^:]+:[^:]*:[0-9]{4}:' /etc/passwd # 5  exactly 4 digits in field 3
```

The `2>/dev/null` is to hide the "permission denied" noise from files grep can't read.

> Basic regex (BRE — default) treats `+ ? { } ( ) |` as literals; you'd write `\{4\}`. Extended (ERE — `grep -E`) treats them as meta. **Use `-E`.** Life is shorter.

---

## Exercise 2 — sed substitutions

```bash
echo "Hello World" | sed 's/World/Linux/'    # 1
sed 's/old/new/g' file.txt                   # 2  /g = all occurrences per line
sed -i 's/old/new/g' file.txt                # 3  -i = in-place (use -i.bak to keep a backup)
sed '/^$/d' file.txt                         # 4  delete blank lines
sed -n '10,20p' file.txt                     # 5  -n = silent default; p = print
```

**Footgun:** `sed -i` on macOS/BSD requires an argument: `sed -i '' '…'`. On GNU sed it's bare `-i`. Portability matters if you ship scripts.

---

## Exercise 3 — awk fundamentals

```bash
awk -F: '{print $1}' /etc/passwd
awk -F: '$7 == "/bin/bash" {print $1}' /etc/passwd
awk -F: '{print $1 "\t" $6}' /etc/passwd
awk -F: '$3 >= 1000 {count++} END {print count}' /etc/passwd
awk '{sum+=$2} END {print sum}' fruit.txt
```

awk's mental model: **pattern { action }**. Either can be omitted. Pattern alone = print matches. Action alone = run for every line. `END { … }` runs once after EOF.

> Built-ins worth knowing: `NR` (current line number), `NF` (number of fields), `FS` (input field sep), `OFS` (output field sep).

---

## Exercise 4 — The classic incantation, decoded

```
cat file.txt              → emit lines
| tr -s '[:space:]' '\n'  → squash runs of whitespace → one word per line
| tr -d '[:punct:]'       → drop punctuation
| tr A-Z a-z              → lowercase
| sort                    → group identical words together
| uniq -c                 → collapse duplicates, prefix with count
| sort -rn                → sort by that count, descending
| head -10                → top 10
```

`uniq` only collapses **adjacent** duplicates — that's why `sort` must come first.

This same 6-step shape (split → normalize → sort → uniq -c → sort -rn → head) is how you analyze 90% of log files. Memorize the shape.

---

## Exercise 5 — find + xargs

```bash
find /var/log -name "*.log" 2>/dev/null | xargs wc -l
find ~ -name "*.txt" -print0 | xargs -0 grep -l "TODO"
```

Why `-print0` / `-0`: filenames can legally contain spaces, newlines, even quotes. Default `xargs` splits on whitespace → it would treat `My File.txt` as two arguments. `-print0` separates with the null byte (a character not allowed in filenames), `-0` tells xargs to expect that.

> Modern alternative: `find … -exec grep -l TODO {} +` does the same job without xargs, and `+` (vs `\;`) batches files.

---

## Exercise 6 — Batch rename

Bash parameter-expansion approach:

```bash
for f in IMG_*.jpg; do
  mv -- "$f" "${f/IMG_/photo_}"
done
```

`mv --` to stop weird filenames being interpreted as flags. `${f/IMG_/photo_}` replaces the first match.

With `rename` (Perl version, common on Debian):

```bash
rename 's/^IMG_/photo_/' IMG_*.jpg
```

`util-linux rename` (the simpler version) has a different syntax: `rename IMG_ photo_ IMG_*.jpg`. Two tools with the same name — read your `man rename` first.

---

## Exercise 7 — Failed-login top 10

```
sudo grep "Failed password" /var/log/auth.log
  | awk '{print $11}'      # field 11 = source IP in default sshd log format
  | sort
  | uniq -c
  | sort -rn
  | head
```

Same 6-step shape as Exercise 4 — pull out a field, sort, count, sort by count, head. Once you see it once, you see it everywhere.

Modern systems may not have `auth.log`; use journalctl:

```bash
sudo journalctl _COMM=sshd | grep "Failed password" | awk '{print $11}' | sort | uniq -c | sort -rn
```

---

## Regex survival kit

```
.      any single char (not newline)
*      0 or more of previous
+      1 or more         (ERE / Perl)
?      0 or 1            (ERE / Perl)
^foo   line starts with
foo$   line ends with
[abc]  any of a,b,c        [^abc] = none of
[0-9]  digit               \d in PCRE
\s \w  whitespace, word char (PCRE / grep -P)
(a|b)  alternation         (ERE)
{n,m}  repeat n to m       (ERE)
\1 \2  backrefs (in replacement: \1)
```

Engine flavors that bite you:
- `grep` (BRE) vs `grep -E` (ERE) vs `grep -P` (Perl, slower, more features)
- `sed` is BRE by default; `sed -E` for ERE
- `awk` uses ERE-ish (POSIX)

---

## Common pitfalls

- **`grep -i foo *`** when no files match: glob fails. Use `grep -ri foo .` instead.
- **`uniq` without sort first.** Returns "almost nothing" because only adjacent duplicates collapse.
- **`awk '{print $1}'` on whitespace-separated input + commas.** awk's default FS is "any run of whitespace" — set `-F,` for CSV.
- **`sed -i` without a backup on irreplaceable files.** Use `sed -i.bak …` while learning.
- **Regex greediness.** `.*` matches as much as possible. Use `.*?` (PCRE) or `[^x]*x` to limit.

→ [Module 08](../../module-08-users-and-groups/README.md)
