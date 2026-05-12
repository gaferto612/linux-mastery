# Module 15 — Systems Programming Intro

**Phase:** Systems programming · **Time:** ~2 weeks · **Prereq:** Module 14

---

**In this module:** [📋 Exercises](exercises/README.md) · [✅ Solutions](solutions/README.md) · [📝 Notes](notes/my-notes.md)

## 🚪 The kernel / user-space boundary

```
   ┌──────────────────────────────────────────────────┐
   │              USER SPACE                          │
   │   your_program  ←→  libc (printf, fopen, ...)    │
   │                          │                       │
   │                          │ syscall instruction   │
   ╞══════════════════════════╪═══════════════════════╡
   │                          ▼                       │
   │              KERNEL SPACE                        │
   │   read()  write()  open()  fork()  execve() ...  │
   │   │                                              │
   │   ▼  drivers, schedulers, filesystems, network   │
   │   hardware                                       │
   └──────────────────────────────────────────────────┘
```

```
   man 1 ls     →  the command
   man 2 read   →  a SYSCALL          (kernel interface)
   man 3 printf →  a LIBRARY function (in libc, runs in user space)
```

## 🔬 strace: watching the boundary

```
   ./hello             libc              Kernel
      │                  │                  │
      │── printf("hi\n")─▶                  │
      │                  │ format string    │
      │                  │── write(1,"hi\n",3) ─▶
      │                  │◀───── 3 ─────────│
      │◀── returns 3 ────│                  │
      │                                     │
   Note: strace shows the write() call, not the printf().
```

## 🧠 Process memory layout

```
   high addresses
   ┌──────────────────┐
   │   stack ▼ grows  │   local vars, function calls
   ├──────────────────┤
   │                  │
   │     (free)       │
   │                  │
   ├──────────────────┤
   │   heap  ▲ grows  │   malloc()'d memory
   ├──────────────────┤
   │   bss            │   zero-init globals
   │   data           │   initialized globals
   │   text           │   the program code (read-only)
   └──────────────────┘
   low addresses
```

---

## What you'll learn

- C basics for Linux: compile, link, run
- The system call concept — the kernel/user-space boundary
- `strace` — see every syscall a program makes
- Reading man pages section 2 (system calls) and section 3 (library functions)
- Memory model basics: stack, heap, text, data

## Readings

| Priority | Book | Chapter |
|---|---|---|
| Required | **TLPI** | Ch. 1 — History and Standards (skim) |
| Required | **TLPI** | Ch. 2 — Fundamental Concepts |
| Required | **TLPI** | Ch. 3 — System Programming Concepts |
| Recommended | **HLW** | Ch. 1 (revisit with new eyes) |

## Note

This is where things shift from "use Linux" to "understand Linux from the inside." You'll write small C programs. If you've never written C, that's fine — these will be small.

**You don't need to become a C programmer.** You need to read enough C to understand the kernel's interface to programs.

## Key concepts

1. **A system call is the gateway from user space to kernel space.** All "Linux features" (file I/O, processes, networking) ultimately go through syscalls.
2. **`man 2 read` is the read() syscall. `man 3 printf` is the printf() library function.** Two different worlds.
3. **`strace` shows the syscalls a program makes.** It is one of the best learning tools in Linux.
4. **A program is a set of instructions + state.** A process is a running program with memory, file descriptors, signals, etc.

## Exercises

In `exercises/`:
- Compile and run a "Hello, World" in C with `gcc`
- Use `strace` on `ls`, `echo hello`, your "Hello World" — count syscalls
- Read `man 2 intro` and skim the syscall list
- Write a program that opens a file and reads it using the `read()` syscall directly
- Compare strace output of statically vs dynamically linked binaries

## Done when...

- You can compile and run a C program without thinking
- `strace ls` makes some sense
- You can name 5 syscalls and what they do

→ [Module 16](../module-16-processes-and-signals/README.md)

---

**Navigate:** [← Previous module](../module-14-backups-and-automation/README.md) · [🏠 Home](../README.md) · [Next module →](../module-16-processes-and-signals/README.md)
