# Module 17 — Files and I/O (in C)

**Phase:** Systems programming · **Time:** ~3 weeks · **Prereq:** Module 16

## What you'll learn

- File descriptors — what they really are
- `open`, `read`, `write`, `close`, `lseek`, `dup`, `dup2`, `pipe`
- How shell redirection (`>`, `<`, `|`) actually works under the hood
- `stat()` and friends — getting file metadata
- File locking basics

## Readings

| Priority | Book | Chapter |
|---|---|---|
| Required | **TLPI** | Ch. 4 — File I/O: The Universal I/O Model |
| Required | **TLPI** | Ch. 5 — File I/O: Further Details |
| Required | **TLPI** | Ch. 15 — File Attributes |
| Recommended | **TLPI** | Ch. 44 — Pipes and FIFOs |

## Key concepts

1. **A file descriptor is an integer.** 0 = stdin, 1 = stdout, 2 = stderr. Everything else gets the next free integer when opened.
2. **The kernel maps fd → open file description → inode.** Three levels. Important when you understand `dup`.
3. **`dup2(src, dst)`** makes `dst` refer to the same thing as `src`. This is how shells implement `>` and `<`.
4. **Pipes are pairs of file descriptors.** Reads block until someone writes; writes block when the buffer fills up.
5. **`stat`** is how you get file metadata (permissions, size, timestamps, owner). It's behind `ls -l`.

## Exercises

In `exercises/`:
- Write your own `cat` using `read`/`write`
- Write your own `cp` (yes, you can — and you'll appreciate it)
- Implement output redirection: spawn a child whose stdout goes to a file
- Implement a pipe between two children: `ls | grep foo`
- Use `stat()` to write your own (simple) `ls -l`

## Done when...

- You can explain how `>` works at the syscall level
- You can implement a pipe in C
- You understand why "everything is a file" isn't just a slogan

→ [Module 18](../module-18-security-fundamentals/README.md)
