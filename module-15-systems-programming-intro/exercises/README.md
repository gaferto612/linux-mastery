# Module 15 — Exercises

## Setup

```bash
sudo apt install build-essential strace gdb
gcc --version
```

## Exercise 1 — Hello, C

Create `hello.c`:

```c
#include <stdio.h>

int main(void) {
    printf("Hello, Linux!\n");
    return 0;
}
```

Compile and run:

```bash
gcc -Wall -o hello hello.c
./hello
```

## Exercise 2 — Look inside

```bash
file hello                   # ELF info
ldd hello                    # shared libraries it depends on
size hello                   # text/data/bss sizes
```

## Exercise 3 — strace your program

```bash
strace ./hello
```

Way more output than you expected, right? Most of that is the dynamic linker setting up libc before your `main()` runs.

Now: `strace -e trace=write ./hello` — only `write` syscalls. See where your "Hello" actually goes out.

## Exercise 4 — strace common commands

```bash
strace -c ls /              # -c gives a summary
strace -c echo hello
strace -c cat /etc/hostname
```

Which is the most common syscall? (Probably `mmap` or `read`.) Read about the top 3 you see.

## Exercise 5 — Read a file by syscall

Create `readfile.c`:

```c
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>

int main(int argc, char *argv[]) {
    if (argc < 2) {
        fprintf(stderr, "Usage: %s <file>\n", argv[0]);
        return 1;
    }

    int fd = open(argv[1], O_RDONLY);
    if (fd < 0) { perror("open"); return 1; }

    char buf[256];
    ssize_t n;
    while ((n = read(fd, buf, sizeof(buf))) > 0) {
        write(STDOUT_FILENO, buf, n);
    }
    close(fd);
    return 0;
}
```

```bash
gcc -Wall -o readfile readfile.c
./readfile /etc/hostname
strace ./readfile /etc/hostname
```

Notice: `open`, `read`, `write`, `close` — these are the syscalls behind every file operation in Linux.

## Exercise 6 — Man page archaeology

```bash
man 2 open                   # the open syscall
man 2 read                   # read
man 2 write
man 2 close
man 2 intro                  # intro to syscalls
```

Pick one. Read the full man page. Note the return values and errno.

## Exercise 7 — Static vs dynamic

```bash
gcc -static -o hello-static hello.c
ls -l hello hello-static          # static is huge
ldd hello                         # has dependencies
ldd hello-static                  # none
strace -c ./hello
strace -c ./hello-static          # different syscall pattern
```

## Reflection

Why does the simplest C program already make dozens of syscalls before printing anything? (Hint: shared library loading.)
