# Module 01 — Terminal Basics

**Phase:** Foundations
**Estimated time:** 2 weeks (~6 hours)
**Prerequisites:** None — you start here.

---

## 🖥️ How your keystrokes become actions

```
You              Terminal Emulator        Bash (the shell)        Kernel
 │                       │                        │                  │
 │  types "ls /etc"      │                        │                  │
 │──────────────────────>│                        │                  │
 │                       │   line of text         │                  │
 │                       │───────────────────────>│                  │
 │                       │                        │ parse & expand   │
 │                       │                        │──────┐           │
 │                       │                        │<─────┘           │
 │                       │                        │ execve("/usr/bin/ls", ["ls","/etc"])
 │                       │                        │─────────────────>│
 │                       │        program output                     │
 │                       │<──────────────────────────────────────────│
 │   pixels on screen    │                                           │
 │<──────────────────────│                                           │
```

1. You type `ls /etc` into the terminal emulator.
2. The terminal sends the line of text to the shell (bash).
3. The shell parses and expands the line.
4. The shell calls the kernel: `execve("/usr/bin/ls", ["ls","/etc"])`.
5. The kernel runs the program and sends output back to the terminal.
6. The terminal draws pixels on your screen.

> **Mental model:** the shell is just another program. Its only job is to read what you type, turn it into a syscall, and print the result.

## 🌳 The filesystem at a glance

```
/
├── bin/   → essential commands (ls, cp, mv...)
├── boot/  → kernel + boot loader files
├── dev/   → devices as files (/dev/sda, /dev/null)
├── etc/   → system config (text files!)
├── home/  → user homes (you live here)
│   └── you/
├── proc/  → live kernel info ("virtual" files)
├── tmp/   → scratch space, wiped on reboot
├── usr/   → most programs and their data
└── var/   → things that change (logs, mail, caches)
```

---

## What you'll learn

By the end of this module, you'll be able to:

- Navigate the Linux filesystem from the terminal without using a file manager
- List, view, copy, move, and delete files using commands
- Read `man` pages and find help for any command
- Use shell shortcuts that make you 5× faster (history, tab completion, wildcards)
- Understand what a "shell" actually is and why it matters

---

## Readings

Pick the readings that match your appetite. Minimum is fine. If a topic confuses you, read it from multiple books — different authors explain it differently.

| Priority | Book | Chapter |
|---|---|---|
| Required | **HLW** | Ch. 1 — The Big Picture |
| Required | **HLW** | Ch. 2 — Basic Commands and Directory Hierarchy (§2.1–2.10) |
| Recommended | **LCLSB** | Ch. 1 — Starting with Linux Shells |
| Recommended | **LCLSB** | Ch. 3 — Basic Bash Shell Commands |
| Optional deep-dive | **ULSAH** | Ch. 1 — Where to Start |

---

## Suggested weekly plan

### Week 1 — Reading and orientation
- Read HLW Ch. 1 and 2.
- Boot up your lab VM. Open a terminal.
- Just *play* — `ls`, `cd`, `pwd`. Look around. Don't break anything yet.
- Start your notes file in `notes/`.

### Week 2 — Exercises and mastery
- Do all exercises in `exercises/`.
- Check yourself against `solutions/`.
- Write a short summary in your notes: "Three things I didn't know two weeks ago."
- Commit your work to git.
- Tick the box in `progress/tracker.md`.

---

## Key concepts to understand (not memorize)

These are the *ideas*. The commands are vocabulary, but these are the grammar.

1. **Everything is a path.** A file, a device, even a process — all addressable via the filesystem.
2. **The shell is a program.** It reads what you type, parses it, runs it. That's it. There's no magic.
3. **Commands are just programs.** `ls` is a file at `/usr/bin/ls`. You can find it. You can replace it.
4. **The current directory is state.** You're always *somewhere*. `pwd` tells you where.
5. **Standard input, output, error.** Three streams. Almost everything in Linux flows through them. (You'll deepen this in later modules.)

---

## Commands you should be able to use without thinking by the end

`pwd`, `cd`, `ls`, `ls -la`, `cat`, `less`, `head`, `tail`, `cp`, `mv`, `rm`, `mkdir`, `rmdir`, `touch`, `man`, `which`, `whoami`, `clear`, `echo`, `history`

Plus shortcuts: **Tab** (autocomplete), **↑/↓** (history), **Ctrl-C** (cancel), **Ctrl-L** (clear), **Ctrl-R** (search history).

---

## When you finish

1. Complete every exercise in `exercises/`.
2. Self-check against `solutions/`.
3. Fill in your notes — at minimum: what surprised you, what's still fuzzy.
4. Commit.
5. Update `progress/tracker.md`.
6. → Move on to [Module 02](../module-02-files-and-permissions/README.md).

---

**Navigate:** — · [🏠 Home](../README.md) · [Next module →](../module-02-files-and-permissions/README.md)
