# Module 17 — Solutions

## Exercise 1 — mycat

```c
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>

int main(int argc, char *argv[]) {
    int fd = STDIN_FILENO;
    if (argc > 1) {
        fd = open(argv[1], O_RDONLY);
        if (fd < 0) { perror("open"); return 1; }
    }
    char buf[4096];
    ssize_t n;
    while ((n = read(fd, buf, sizeof buf)) > 0) {
        ssize_t off = 0;
        while (off < n) {
            ssize_t w = write(STDOUT_FILENO, buf + off, n - off);
            if (w < 0) { perror("write"); return 1; }
            off += w;            // write may write less than asked — keep going
        }
    }
    if (n < 0) { perror("read"); return 1; }
    if (fd != STDIN_FILENO) close(fd);
    return 0;
}
```

Things to internalize:

- **`read` returns a short count.** Asked for 4096, may get 100. That's not an error — it's I/O semantics.
- **`write` returns a short count too.** Loop until you've written everything you read.
- **`read` returns 0 = EOF.** `< 0` = error.
- **Standard FD constants** — `STDIN_FILENO`, `STDOUT_FILENO`, `STDERR_FILENO` are clearer than magic 0/1/2.

Buffer-size economics: 4096 is one page on most systems. Bigger isn't faster past a point — at some size you're cache-thrashing.

---

## Exercise 2 — mycp

```c
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>
#include <sys/stat.h>

int main(int argc, char *argv[]) {
    if (argc != 3) { fprintf(stderr, "usage: %s src dst\n", argv[0]); return 2; }

    int in  = open(argv[1], O_RDONLY);
    if (in < 0) { perror(argv[1]); return 1; }

    int out = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (out < 0) { perror(argv[2]); close(in); return 1; }

    char buf[4096];
    ssize_t n;
    while ((n = read(in, buf, sizeof buf)) > 0) {
        ssize_t off = 0;
        while (off < n) {
            ssize_t w = write(out, buf + off, n - off);
            if (w < 0) { perror("write"); return 1; }
            off += w;
        }
    }
    close(in);
    close(out);
    return n < 0 ? 1 : 0;
}
```

`open` flag breakdown:

```
O_RDONLY / O_WRONLY / O_RDWR   pick one (mode)
O_CREAT                         create if missing       (then mode arg matters)
O_TRUNC                         truncate to 0 if exists
O_EXCL                          (with O_CREAT) fail if exists
O_APPEND                        every write goes to end of file (atomic)
```

The trailing `0644` is the **permissions** to use IF the file is being created (and gets ANDed with umask). Ignored otherwise.

For fast bulk copies, modern Linux has `copy_file_range(2)` — kernel-level, no user-space round-trip. Real `cp` uses it.

---

## Exercise 3 — Redirect stdout to a file

```c
pid_t pid = fork();
if (pid == 0) {
    int fd = open("output.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    dup2(fd, STDOUT_FILENO);    // make fd 1 point to the same open-file entry as fd
    close(fd);                  // we don't need the original fd anymore
    execlp("ls", "ls", "-la", "/", (char *)NULL);
    perror("exec"); return 1;
}
wait(NULL);
```

Step-by-step in the child:

```
1. open("output.txt", …)  →  some fd, say 3
2. dup2(3, 1)              →  fd 1 (stdout) now points to output.txt
                              fd 3 ALSO points to output.txt (still open)
3. close(3)                →  only fd 1 left pointing there
4. execlp("ls", …)         →  ls runs, its stdout writes go to output.txt
```

This is exactly what bash does when you type `ls > output.txt`. The `>` is not a feature of `ls`; ls just writes to stdout. The shell rewires stdout before exec.

**Why `dup2` and not `dup`:** `dup2(src, dst)` guarantees the result is `dst`. `dup(src)` returns the lowest free fd, which is not predictable.

---

## Exercise 4 — Implement a pipe

```c
int p[2];
pipe(p);   // p[0] = read end, p[1] = write end (mnemonic: 0 in, 1 out... wait, opposite)
           // mnemonic: "0 like stdin (you read), 1 like stdout (you write)"

if (fork() == 0) {
    // child 1: "ls" — writes to the pipe
    dup2(p[1], STDOUT_FILENO);
    close(p[0]); close(p[1]);          // close BOTH; we replaced stdout already
    execlp("ls", "ls", "/", (char *)NULL);
}
if (fork() == 0) {
    // child 2: "grep e" — reads from the pipe
    dup2(p[0], STDIN_FILENO);
    close(p[0]); close(p[1]);
    execlp("grep", "grep", "e", (char *)NULL);
}
close(p[0]); close(p[1]);              // parent must also close, or grep never sees EOF
wait(NULL); wait(NULL);
```

**Why the parent must close both ends.** A pipe's "writer closed" signal (EOF on the reader) only fires when *every* fd pointing to the write end is closed. If the parent leaves `p[1]` open, grep waits forever for more input.

This is the #1 pipe bug. The rule: in each process, close every pipe fd you're not actively using.

---

## Exercise 5 — myls

```c
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>
#include <sys/stat.h>
#include <unistd.h>
#include <pwd.h>
#include <grp.h>
#include <time.h>
#include <string.h>

static void print_mode(mode_t m) {
    char s[11] = "----------";
    if (S_ISDIR(m))  s[0] = 'd';
    if (S_ISLNK(m))  s[0] = 'l';
    if (m & S_IRUSR) s[1]='r'; if (m & S_IWUSR) s[2]='w'; if (m & S_IXUSR) s[3]='x';
    if (m & S_IRGRP) s[4]='r'; if (m & S_IWGRP) s[5]='w'; if (m & S_IXGRP) s[6]='x';
    if (m & S_IROTH) s[7]='r'; if (m & S_IWOTH) s[8]='w'; if (m & S_IXOTH) s[9]='x';
    printf("%s ", s);
}

int main(int argc, char *argv[]) {
    const char *dir = argc > 1 ? argv[1] : ".";
    DIR *d = opendir(dir);
    if (!d) { perror(dir); return 1; }

    struct dirent *e;
    while ((e = readdir(d)) != NULL) {
        if (e->d_name[0] == '.') continue;     // skip dotfiles for now
        char path[4096];
        snprintf(path, sizeof path, "%s/%s", dir, e->d_name);

        struct stat st;
        if (lstat(path, &st) < 0) { perror(path); continue; }

        print_mode(st.st_mode);
        struct passwd *pw = getpwuid(st.st_uid);
        struct group  *gr = getgrgid(st.st_gid);
        printf("%-8s %-8s %8ld %s\n",
               pw ? pw->pw_name : "?",
               gr ? gr->gr_name : "?",
               (long)st.st_size, e->d_name);
    }
    closedir(d);
    return 0;
}
```

`lstat` (not `stat`) so symlinks themselves are described, not their targets.

`stat` fills a struct with:

```
st_mode    type bits + permission bits
st_uid / st_gid   owner ids → look up names via getpwuid/getgrgid
st_size    bytes
st_mtim    modification time (struct timespec; ns resolution)
st_ino     inode number
st_nlink   number of hard links
st_blocks  512-byte blocks actually allocated (sparse-file detection)
```

You've now reimplemented the essence of `ls -l`. Real ls handles colors, sorting, recursion, ACLs, locales — but the core is here.

---

## What you understand now that you didn't before

A pipeline like `ls / | grep e > out.txt 2>&1` is, mechanically:

```c
int p[2]; pipe(p);

if (fork() == 0) {                          // "ls"
    dup2(p[1], 1); close(p[0]); close(p[1]);
    execvp("ls", (char *[]){"ls","/",NULL});
}
if (fork() == 0) {                          // "grep e > out.txt 2>&1"
    int fd = open("out.txt", O_WRONLY|O_CREAT|O_TRUNC, 0644);
    dup2(p[0], 0);                          // grep reads pipe
    dup2(fd, 1);                            // grep stdout → out.txt
    dup2(1, 2);                             // stderr → same as stdout
    close(p[0]); close(p[1]); close(fd);
    execvp("grep", (char *[]){"grep","e",NULL});
}
close(p[0]); close(p[1]);
wait(NULL); wait(NULL);
```

Bash is not magic. It's ~50 lines of fork/exec/dup2 wrapped in a parser.

---

## Common pitfalls

- **Forgetting to close the pipe in the parent** → reader never sees EOF.
- **`dup2` confusion:** `dup2(oldfd, newfd)` makes `newfd` point to `oldfd`'s target. The order trips people.
- **Short reads/writes treated as errors.** They're not. Loop until you've moved every byte.
- **Mixing buffered stdio (`printf`) with raw I/O (`write`).** Buffers don't see each other → output interleaves weirdly. Pick one within a single fd.
- **Not checking `open`'s return value.** -1 on failure; treating it as a fd → reads/writes to whatever was at fd -1 → wild crashes.
- **Off-by-one in `read(fd, buf, sizeof buf - 1)`** when you want a null terminator. `read` doesn't add one — you do.

→ [Module 18](../../module-18-security-fundamentals/README.md)
