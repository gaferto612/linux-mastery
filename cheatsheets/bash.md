# Bash Cheatsheet

## Navigation
| Command | What it does |
|---|---|
| `pwd` | Print current directory |
| `cd <dir>` | Change directory |
| `cd` or `cd ~` | Go home |
| `cd -` | Go to previous directory |
| `cd ..` | Up one level |

## Files and directories
| Command | What it does |
|---|---|
| `ls -la` | List all, long format |
| `ls -lh` | Human-readable sizes |
| `ls -lt` | Sort by modification time |
| `cp src dst` | Copy file |
| `cp -r src dst` | Copy directory |
| `mv src dst` | Move / rename |
| `rm file` | Delete file |
| `rm -rf dir` | Delete directory recursively (be careful!) |
| `mkdir -p a/b/c` | Make nested dirs |
| `touch file` | Create empty file / update timestamp |

## Viewing files
| Command | What it does |
|---|---|
| `cat file` | Print whole file |
| `less file` | Scrollable viewer (q to quit) |
| `head -n 20 file` | First 20 lines |
| `tail -n 20 file` | Last 20 lines |
| `tail -f file` | Follow new lines as they appear |
| `wc -l file` | Count lines |

## Searching
| Command | What it does |
|---|---|
| `grep 'pattern' file` | Search in file |
| `grep -r 'pat' dir` | Recursive |
| `grep -i 'pat' file` | Case-insensitive |
| `grep -v 'pat' file` | Lines NOT matching |
| `find /path -name '*.txt'` | Find by name |
| `find /path -type f -size +10M` | Find by size |
| `find /path -mtime -7` | Modified in last 7 days |

## Pipes and redirection
| Operator | What it does |
|---|---|
| `\|` | Pipe stdout of left into stdin of right |
| `>` | Redirect stdout to file (overwrite) |
| `>>` | Redirect stdout to file (append) |
| `<` | Use file as stdin |
| `2>` | Redirect stderr |
| `&>` | Redirect both stdout and stderr |
| `2>&1` | Merge stderr into stdout |

## Variables
```bash
name="Alice"
echo "$name"          # quote to be safe
echo "${name}_suffix" # use braces when ambiguous
result=$(command)     # command substitution
```

## Conditionals
```bash
if [ -f file.txt ]; then echo "exists"; fi
if [ "$x" -gt 5 ]; then echo "big"; fi
if [[ "$s" == *foo* ]]; then echo "match"; fi  # [[ ]] is bash-only, supports more
```

## Loops
```bash
for f in *.txt; do echo "$f"; done
for i in {1..10}; do echo "$i"; done
while read -r line; do echo "$line"; done < file.txt
```

## Job control
| Key | What it does |
|---|---|
| Ctrl-C | Send SIGINT (interrupt) |
| Ctrl-Z | Suspend foreground process |
| Ctrl-D | EOF (end input) |
| `bg` | Resume suspended job in background |
| `fg` | Bring background job to foreground |
| `jobs` | List background jobs |
| `&` (suffix) | Run command in background |

## Shortcuts
| Key | What it does |
|---|---|
| Tab | Autocomplete |
| Ctrl-R | Reverse history search |
| Ctrl-L | Clear screen |
| Ctrl-A | Jump to start of line |
| Ctrl-E | Jump to end of line |
| Ctrl-U | Delete from cursor to start |
| Ctrl-K | Delete from cursor to end |
| Ctrl-W | Delete word before cursor |
| !! | Last command |
| !$ | Last argument of last command |

## Safe script header

```bash
#!/bin/bash
set -euo pipefail
IFS=$'\n\t'
```

## Useful one-liners

```bash
# 10 biggest files in a tree
find . -type f -exec du -h {} + | sort -rh | head

# Count files by extension
find . -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn

# Replace in all matching files
find . -name '*.txt' -exec sed -i 's/old/new/g' {} +

# Disk usage by directory
du -h --max-depth=1 | sort -rh
```

---

[← Back to course home](../README.md)
