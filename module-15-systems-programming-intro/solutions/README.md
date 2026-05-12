# Module 15 — Solutions

## Setup

```bash
sudo apt install build-essential strace gdb
gcc --version
```

`build-essential` is a meta-package: pulls in gcc, g++, make, libc headers, and a few build tools.

---

## Exercise 1 — Hello, C

```c
#include <stdio.h>

int main(void) {
    printf("Hello, Linux!\n");
    return 0;
}
```

```bash
gcc -Wall -Wextra -o hello hello.c
./hello
```

`-Wall -Wextra` turns on a wider net of warnings. Treat warnings as errors as you learn: `-Werror`. Cheap habit, prevents bugs.

What `gcc` actually did:

1. **Preprocess** — expand `#include`, macros (`gcc -E`)
2. **Compile** — C → assembly (`gcc -S`)
3. **Assemble** — assembly → object code (`gcc -c`)
4. **Link** — object + libc → executable

You can see each stage with the corresponding flag.

---

## Exercise 2 — Look inside

```bash
file hello
# hello: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, …

ldd hello
# linux-vdso.so.1
# libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6
# /lib64/ld-linux-x86-64.so.2          ← the dynamic linker

size hello
#    text    data     bss     dec     hex filename
#    1234     512       8    1754     6da hello
```

- **text** — code (instructions). Read-only, often shared between copies of the same program.
- **data** — initialized globals.
- **bss** — uninitialized globals (zero-filled at load time).
- **vdso** — kernel-mapped page for fast syscalls (`gettimeofday`, `clock_gettime`).

---

## Exercise 3 — strace your program

```bash
strace ./hello
```

You'll see a wall of output. The shape, roughly:

```
execve("./hello", …)
brk(NULL)                              # heap setup
mmap(…)                                # map libc, ld-linux into memory
openat(…, "/etc/ld.so.cache", …)       # find shared libs
openat(…, "/lib/x86_64-linux-gnu/libc.so.6", …)
mprotect(…)                            # set page permissions
…                                       # ← all of the above is the dynamic linker
write(1, "Hello, Linux!\n", 14) = 14   # ← finally your code
exit_group(0)
```

Narrow to just your code's syscalls:

```bash
strace -e trace=write ./hello
```

That's the one line your program is really responsible for. Everything before it was libc/ld setup.

---

## Exercise 4 — strace common commands

```bash
strace -c ls /
strace -c echo hello
strace -c cat /etc/hostname
```

`-c` summarizes per syscall. Typical chart-toppers:

- `mmap` — every shared lib gets mapped
- `read` / `write` — actual I/O
- `openat` / `close` — every file
- `mprotect` — flipping page permissions
- `newfstatat` — stat'ing files (ls leans on this)

Read `man 2 mmap` once and you understand a third of the syscalls modern programs make.

---

## Exercise 5 — Read a file by syscall

The C file from the exercise compiles cleanly. Key things to notice:

```c
int fd = open(argv[1], O_RDONLY);    // syscall: returns small integer
if (fd < 0) { perror("open"); }      // -1 on error, errno set

ssize_t n;
char buf[256];
while ((n = read(fd, buf, sizeof(buf))) > 0) {   // read returns BYTES read
    write(STDOUT_FILENO, buf, n);                // STDOUT_FILENO is just 1
}
close(fd);
```

The pattern open → loop(read) → close is *every* file-reading program in Linux, including `cat`. You just wrote `cat`.

`perror("open")` prints `"open: No such file or directory"` etc. — it reads the global `errno` set by the failed syscall.

---

## Exercise 6 — Man page archaeology

```bash
man 2 open       # section 2 = SYSCALLS — the kernel interface
man 2 read
man 2 write
man 2 close
man 2 intro      # general intro to syscalls
```

Section guide:

```
1   user commands (ls, grep)
2   syscalls (read, open, fork)
3   library functions (printf, malloc)
4   special files (/dev/null, /dev/tty)
5   file formats (passwd, fstab)
7   miscellaneous / overview (signal, capabilities)
8   admin commands (mount, sshd)
```

When a name exists in multiple sections, `man printf` defaults to section 1 (the command). To get the C function: `man 3 printf`.

---

## Exercise 7 — Static vs dynamic

```bash
gcc -static -o hello-static hello.c
ls -l hello hello-static
# hello         ~16 KB
# hello-static  ~800 KB  (whole libc baked in)

ldd hello
ldd hello-static                # "not a dynamic executable"

strace -c ./hello | wc -l       # ~30 syscalls
strace -c ./hello-static | wc -l  # far fewer — no dynamic linker dance
```

Static binaries: bigger, but self-contained — useful for distributing one-file tools, containers, or running on systems without your libc version.

Dynamic linking wins: shared memory across processes (one libc in RAM, not one per process), security patches to libc apply to every program at once.

---

## Why so many syscalls for "Hello"

The dynamic linker (`ld-linux.so`) runs FIRST, before your `main`:

1. Maps itself into the process.
2. Reads the ELF header → list of needed libraries.
3. Opens each (`openat`), reads the symbol table (`mmap`).
4. Resolves all symbols `printf`, `__libc_start_main`, etc.
5. Sets permissions on each mapped region (`mprotect`).
6. Calls `__libc_start_main`, which sets up stdio buffers, locale, env … then your `main`.

That's why a "do nothing" C program already makes ~30 syscalls.

---

## errno crash course

After a failed syscall:

```c
if (fd < 0) {
    fprintf(stderr, "open failed: %s\n", strerror(errno));
    // or, simpler:
    perror("open");
    return 1;
}
```

`errno` is a global (technically thread-local) integer set by the last failing syscall. `strerror(errno)` turns it into a human string. `perror("msg")` prints `"msg: human string"` to stderr.

Common errnos to recognize:

```
ENOENT     no such file or directory
EACCES     permission denied
EEXIST     file exists (e.g. O_EXCL on create)
EBADF      bad file descriptor (closed or never opened)
EAGAIN     try again later (non-blocking I/O)
EINTR      interrupted by a signal — RETRY the syscall
```

`EINTR` is the one that bites C programmers: a signal during a blocking syscall returns -1 with errno=EINTR. You're supposed to retry, not error out.

---

## Common pitfalls

- **Forgetting `close(fd)`** — file descriptors leak. After ~1024 leaks the process can't open anything.
- **Ignoring return values.** Every syscall can fail. `read` may return less than you asked for (short read). `write` likewise.
- **Treating `errno` as preserved across calls.** Save it: `int e = errno;` before any other call that could clobber it.
- **Mixing `printf` and `write` without `fflush`.** stdio buffers; your output may interleave weirdly.
- **`gcc` without `-Wall`.** Modern gcc catches almost all classic C mistakes if you let it.

→ [Module 16](../../module-16-processes-and-signals/README.md)
