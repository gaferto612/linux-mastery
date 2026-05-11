# Module 01 — Solutions

These are the answers, but more importantly **the reasoning**. If your command worked but you don't understand why, read the explanation.

---

## Exercise 1 — Where am I, what's here?

```bash
pwd                    # 1. print working directory
ls -la                 # 2. -l = long format, -a = include hidden (dotfiles)
cd                     # 4. with no args, cd takes you home
pwd                    # 5. confirm
```

**Why `cd` with no arguments?** Bash treats `cd` alone as `cd $HOME`. Faster than typing `cd ~` or `cd /home/yourname`.

---

## Exercise 2 — Create and destroy

```bash
mkdir ~/linux-practice
cd ~/linux-practice
touch notes.txt todo.txt journal.txt
ls -lh                 # -h = human readable sizes (though these are empty)
rm todo.txt
mv journal.txt diary.txt
ls
```

**Note:** `touch` creates an empty file *or* updates the timestamp of an existing one. Surprises beginners later — keep it in mind.

---

## Exercise 3 — Reading files

```bash
cat /etc/hostname
head -n 5 /etc/passwd
tail -n 5 /etc/passwd
less /etc/passwd       # /root then Enter to search, q to quit
```

**`cat` vs `less`:** `cat` dumps the whole file at once (great for small files, terrible for huge ones). `less` lets you scroll. Rule of thumb: if the file might be more than a screen, use `less`.

---

## Exercise 4 — Copying and moving

```bash
cp /etc/hostname ~/linux-practice/
cd ~/linux-practice
mkdir backup
mv hostname backup/
cp -r backup/ backup2/    # -r = recursive (required for directories)
ls backup/ backup2/
```

**Why `-r`?** By default `cp` refuses to copy directories. The `-r` (or `-R`) flag tells it: "yes, recurse into this folder and copy everything inside too."

---

## Exercise 5 — The mighty `man`

```bash
man ls
# inside man: type /modification then Enter to search
# answer: -t  (sort by modification time, newest first)
# answer: -h  (human readable)
ls -lht /etc           # combined
q                      # quit man
```

**Tip:** man pages are searchable with `/pattern`, navigate with arrow keys or space, quit with `q`. Same controls as `less` (because man pages *use* `less` under the hood).

---

## Exercise 6 — Where does a command live?

```bash
which ls               # likely /usr/bin/ls or /bin/ls
which cat              # likely /usr/bin/cat
which bash             # likely /usr/bin/bash or /bin/bash
ls -la /usr/bin/ls     # check what kind of file it is
```

**What you might notice:** on many modern systems, `/bin` is a symlink to `/usr/bin`. So `/bin/ls` and `/usr/bin/ls` are the same file. You'll learn about symlinks in Module 2.

---

## Exercise 7 — History and shortcuts

```bash
history | tail -n 20
# ↑ arrow re-pulls previous commands
# Ctrl-R = reverse search through history (very powerful, practice this!)
# Tab on "cd /et" autocompletes to "cd /etc/"
# Ctrl-L clears the screen visually but keeps your history; "clear" does the same
```

**Pro tip:** Ctrl-R is the single shortcut that separates beginners from intermediates. Use it every day for a week and it'll become muscle memory.

---

## Exercise 8 — Wildcards

```bash
ls *.txt               # 1. all .txt files
ls file*               # 2. anything starting with "file"
ls ?.txt               # 3. exactly one character + .txt
rm *.md                # 4. all .md files
```

**Important concept:** The shell expands wildcards *before* the command runs. So `ls *.txt` actually becomes `ls file1.txt file2.txt file3.txt` before `ls` ever sees it. This means `ls` doesn't "know about" wildcards — the shell handles them. You'll appreciate this distinction later.

---

## Common mistakes from this module

- **Forgetting `-r` when copying folders.** `cp folder/ dest/` will fail or behave oddly. Use `cp -r`.
- **Using `cat` on huge files.** It works but floods your terminal. Always `less` for unknown sizes.
- **Confusing `mv` with rename.** There is no `rename` command in most cases — `mv` does both.
- **`rm` doesn't ask before deleting.** There is no trash bin. Be careful, especially with wildcards.

---

## You're done with Module 01 when...

- You can do all 8 exercises from memory.
- `man <command>` doesn't feel scary anymore.
- You catch yourself using **Tab** without thinking.

→ On to [Module 02 — Files and Permissions](../../module-02-files-and-permissions/README.md)
