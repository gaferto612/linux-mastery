# Module 01 — Exercises

Work through these in order. For each, **type the commands yourself** — don't copy-paste.
Take notes when something surprises you.

If you get stuck for more than 10 minutes, check `solutions/` — but really try first.

---

## Exercise 1 — Where am I, what's here?

1. Open a terminal.
2. Print the current directory.
3. List everything in it, including hidden files, in long format.
4. Move to your home directory using the shortest possible command.
5. Confirm where you are.

---

## Exercise 2 — Create and destroy

1. In your home directory, create a folder called `linux-practice`.
2. Inside it, create three files: `notes.txt`, `todo.txt`, `journal.txt`.
3. List the contents of `linux-practice` showing file sizes and modification dates.
4. Delete `todo.txt`.
5. Rename `journal.txt` to `diary.txt`.
6. Verify only `notes.txt` and `diary.txt` remain.

---

## Exercise 3 — Reading files

1. Display the contents of `/etc/hostname`.
2. Display the first 5 lines of `/etc/passwd`.
3. Display the last 5 lines of `/etc/passwd`.
4. Open `/etc/passwd` in a scroller you can quit with `q` (hint: `less`).
5. Inside `less`: search for the word `root` by pressing `/root` then Enter. Press `q` to quit.

---

## Exercise 4 — Copying and moving

1. Copy `/etc/hostname` into your `linux-practice` folder.
2. Inside `linux-practice`, make a new folder called `backup`.
3. Move the copied `hostname` file into `backup/`.
4. Copy the entire `backup/` folder to a new folder called `backup2/`. (Hint: you'll need a flag for recursive copy.)
5. List the contents of both `backup/` and `backup2/` to confirm.

---

## Exercise 5 — The mighty `man`

1. Open the manual page for `ls`.
2. Find the flag that sorts files by modification time. Write it in your notes.
3. Find the flag that gives "human readable" file sizes (KB, MB instead of bytes).
4. Combine `-l`, the human-readable flag, and the sort-by-time flag into one `ls` command, and run it on `/etc`.
5. Quit the man page.

---

## Exercise 6 — Where does a command live?

1. Find the full path to the `ls` command.
2. Find the full path to `cat`.
3. Find the full path to `bash`.
4. Open the `ls` file's actual location with `ls -la` on it. (Use the path you found.) Is it a regular file, or a link?

---

## Exercise 7 — History and shortcuts

1. Show the last 20 commands you've run.
2. Re-run an old command using the **↑** arrow.
3. Press **Ctrl-R** and start typing `ls` — what happens?
4. Type `cd /et` then press **Tab**. What does the shell complete?
5. Press **Ctrl-L**. Then **clear**. What's the difference? (Trick question. Notice anything?)

---

## Exercise 8 — Wildcards (a teaser)

In `linux-practice/`, create these files using one command each:

```
touch file1.txt file2.txt file3.txt note.md README.md
```

Then:

1. List only the `.txt` files.
2. List only the files starting with `file`.
3. List only files that have exactly one character before `.txt` (e.g., `a.txt`, `1.txt`, but not `aa.txt`). Hint: `?`
4. Delete only the `.md` files in one command.

---

## Cleanup

When you're done, you can delete your `linux-practice` folder — or keep it as your scratch space for later modules.

---

## Reflection (in your notes)

- Which command felt most useful?
- Which one confused you?
- What's one thing you want to try next?
