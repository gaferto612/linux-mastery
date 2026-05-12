# Linux Mastery — A 12-Month Self-Study Course

A structured, beginner-friendly path from "I've opened a terminal" to "I understand how Linux really works."
Built around five classic books, designed for ~3 hours/week.

## Where to start

→ [**Module 01 — Terminal basics**](module-01-terminal-basics/README.md)

Good luck. You'll be surprised how far you get in a year.

---

```
┌───────────────────────────────────────────────────────────────────────┐
│                                                                       │
│   MONTH:  1   2   3   4   5   6   7   8   9   10  11  12              │
│   PHASE: [Foundations ][Boot][ Scripting ][   SysAdmin   ][SysProg][Sec]│
│   YOU →  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  🎓 │
│                                                                       │
└───────────────────────────────────────────────────────────────────────┘
```

## Quick navigation

- [How this course works](#how-this-course-works)
- [Before you start](#before-you-start)
- [The five books](#the-five-books-youll-use)
- [The roadmap](#the-roadmap)
- [How to use each module](#how-to-use-each-module)
- [Cheatsheets](#cheatsheets) · [Progress tracker](progress/tracker.md) · [Setup guide](resources/setup-your-lab.md) · [Further reading](resources/further-reading.md)

---

## Visual roadmap

```
PHASE 1 · Foundations
  01 Terminal  →  02 Files & Perms  →  03 Processes
        │
        ▼
PHASE 2 · Internals
  04 How Linux Boots
        │
        ▼
PHASE 3 · Scripting
  05 Bash Basics  →  06 Bash Advanced  →  07 grep/sed/awk
        │
        ▼
PHASE 4 · System Administration
  08 Users  →  09 Networking  →  10 Storage  →  11 Packages
        →  12 systemd  →  13 Logs  →  14 Backups
        │
        ▼
PHASE 5 · Systems Programming (C)
  15 Syscalls  →  16 fork/exec  →  17 Files & I/O
        │
        ▼
PHASE 6 · Security
  18 Hardening  →  19 Pentesting  →  20 Capstone
```

---

## How this course works

You read **chapters from real books** (the ones you already have), then come here to do **exercises** that lock the knowledge in. Each module is one focused topic, about 2–4 weeks of casual study.

This repo doesn't replace the books — it guides you through them and gives you hands-on practice. You learn Linux by *using* Linux, not by reading about it.

### The five books you'll use

| Short name | Full title |
|---|---|
| **HLW** | How Linux Works — Brian Ward |
| **LCLSB** | Linux Command Line and Shell Scripting Bible — Blum & Bresnahan |
| **TLPI** | The Linux Programming Interface — Michael Kerrisk |
| **ULSAH** | UNIX and Linux System Administration Handbook — Nemeth et al. |
| **MPTG** | Metasploit: The Penetration Tester's Guide — Kennedy et al. |

Throughout the modules, readings are referenced like: `LCLSB ch.3` or `HLW §2.4`.

---

## Before you start

You need a Linux system to practice on. Pick one:

- **Best:** A virtual machine running Ubuntu or Debian (use VirtualBox, UTM on Mac, or VMware). You can break things safely.
- **Good:** WSL2 on Windows (Ubuntu).
- **OK:** A cheap VPS ($5/month from Hetzner, DigitalOcean, etc.) — bonus: you also learn remote work via SSH.
- **Avoid for now:** Using your main daily-driver Linux machine. You'll be experimenting and breaking things on purpose.

See [`resources/setup-your-lab.md`](resources/setup-your-lab.md) for a full setup guide.

---

## The five books at a glance

| Book | Author | Focus |
|---|---|---|
| **HLW** — How Linux Works | Brian Ward | Internals & mental models |
| **LCLSB** — Linux Command Line & Shell Scripting Bible | Blum & Bresnahan | Daily driving the shell |
| **TLPI** — The Linux Programming Interface | Michael Kerrisk | Kernel API in C |
| **ULSAH** — UNIX & Linux SysAdmin Handbook | Nemeth et al. | Running real systems |
| **MPTG** — Metasploit: The Penetration Tester's Guide | Kennedy et al. | Offensive security |

---

## The roadmap

### Phase 1 — Foundations (Months 1–2)
You learn to live in the terminal. By the end you should be comfortable navigating, manipulating files, and not panicking when something looks weird.

- [**Module 01**](module-01-terminal-basics/README.md) — Terminal basics
- [**Module 02**](module-02-files-and-permissions/README.md) — Files and permissions
- [**Module 03**](module-03-processes-and-jobs/README.md) — Processes and jobs

### Phase 2 — How Linux actually works (Month 3)
You stop seeing Linux as magic. You understand what happens between power-on and login prompt.

- [**Module 04**](module-04-how-linux-boots/README.md) — How Linux boots

### Phase 3 — Bash scripting (Months 4–5)
The single most leveraged skill on Linux. Once you can script, you can automate anything.

- [**Module 05**](module-05-shell-scripting-basics/README.md) — Shell scripting basics
- [**Module 06**](module-06-shell-scripting-advanced/README.md) — Shell scripting advanced
- [**Module 07**](module-07-text-processing/README.md) — Text processing (grep, sed, awk)

### Phase 4 — System administration (Months 6–8)
You can now manage a real Linux system: users, networks, disks, services, logs.

- [**Module 08**](module-08-users-and-groups/README.md) — Users and groups
- [**Module 09**](module-09-networking/README.md) — Networking
- [**Module 10**](module-10-storage-and-filesystems/README.md) — Storage and filesystems
- [**Module 11**](module-11-package-management/README.md) — Package management
- [**Module 12**](module-12-services-and-systemd/README.md) — Services and systemd
- [**Module 13**](module-13-logging-and-monitoring/README.md) — Logging and monitoring
- [**Module 14**](module-14-backups-and-automation/README.md) — Backups and automation

### Phase 5 — Systems programming (Months 9–10)
You write C programs that talk to the kernel directly. This is where Linux stops being a black box.

- [**Module 15**](module-15-systems-programming-intro/README.md) — Systems programming intro
- [**Module 16**](module-16-processes-and-signals/README.md) — Processes and signals
- [**Module 17**](module-17-files-and-io/README.md) — Files and I/O

### Phase 6 — Security (Months 11–12)
Now that you understand the system, you can learn how to defend it — and how it gets attacked.

- [**Module 18**](module-18-security-fundamentals/README.md) — Security fundamentals
- [**Module 19**](module-19-intro-to-pentesting/README.md) — Intro to pentesting
- [**Module 20**](module-20-capstone-projects/README.md) — Capstone projects

---

## How to use each module

Every module has the same structure:

```
module-XX-name/
├── README.md          # what you'll learn, readings, weekly plan
├── notes/             # your own notes (template provided)
├── exercises/         # tasks to complete
└── solutions/         # check your work AFTER you try
```

The workflow for each module:

1. **Read** the assigned chapters from the book(s).
2. **Take notes** in `notes/` — your own words, what clicked, what didn't.
3. **Do the exercises** in `exercises/`. Struggle a bit before peeking.
4. **Check** your work against `solutions/`.
5. **Commit** your progress to git. (Yes, you'll learn git through using it.)
6. **Update** [`progress/tracker.md`](progress/tracker.md).

---

## Time commitment

About 3 hours per week, roughly:

- 90 min — reading
- 60 min — exercises
- 30 min — notes and review

If you fall behind: that's fine. This is a marathon. Skip the deadline, not the work.

---

## A few rules

1. **Type every command yourself.** Don't copy-paste from the book. Your fingers learn too.
2. **Read `man` pages.** When a command is new, `man command` is your friend. Even partially. Even if it's confusing.
3. **Break things on purpose.** The VM exists to be destroyed and rebuilt.
4. **Don't memorize. Understand.** If you understand *why*, you'll re-derive the *how*.
5. **Commit often.** Even tiny progress. The graph keeps you honest.

---

## Cheatsheets

Quick references you can keep open while you work:

- [bash](cheatsheets/bash.md)
- [permissions](cheatsheets/permissions.md)
- [networking](cheatsheets/networking.md)
- [systemd](cheatsheets/systemd.md)
- [vim survival](cheatsheets/vim-survival.md)

---

→ Ready to begin? [**Start with Module 01**](module-01-terminal-basics/README.md).
