# Module 17 — Exercises

## Exercise 1 — mycat

`mycat.c`:

```c
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>

int main(int argc, char *argv[]) {
    int fd = 0;  // stdin by default
    if (argc > 1) {
        fd = open(argv[1], O_RDONLY);
        if (fd < 0) { perror("open"); return 1; }
    }
    char buf[4096];
    ssize_t n;
    while ((n = read(fd, buf, sizeof(buf))) > 0) {
        write(1, buf, n);
    }
    if (fd > 0) close(fd);
    return 0;
}
```

`./mycat /etc/hostname` — and you've reimplemented `cat`. Notice: 50 lines or fewer.

## Exercise 2 — mycp

Write `mycp.c` that copies from arg1 to arg2 in 4096-byte chunks. Handle errors. Use `open` with `O_WRONLY | O_CREAT | O_TRUNC` for the destination.

## Exercise 3 — Redirect stdout to a file (in C)

`redirect.c`:

```c
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>
#include <sys/wait.h>

int main(void) {
    pid_t pid = fork();
    if (pid == 0) {
        int fd = open("output.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
        dup2(fd, STDOUT_FILENO);    // now fd 1 (stdout) points to the file
        close(fd);
        execlp("ls", "ls", "-la", "/", (char *)NULL);
        perror("exec"); return 1;
    }
    wait(NULL);
    printf("Done. See output.txt\n");
}
```

This is how `ls > output.txt` is implemented inside the shell. You just wrote a piece of bash.

## Exercise 4 — Implement a pipe

`mypipe.c`:

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main(void) {
    int p[2];
    pipe(p);   // p[0] = read end, p[1] = write end

    if (fork() == 0) {
        // child 1: ls
        dup2(p[1], STDOUT_FILENO);
        close(p[0]); close(p[1]);
        execlp("ls", "ls", "/", (char *)NULL);
    }
    if (fork() == 0) {
        // child 2: grep e
        dup2(p[0], STDIN_FILENO);
        close(p[0]); close(p[1]);
        execlp("grep", "grep", "e", (char *)NULL);
    }
    close(p[0]); close(p[1]);
    wait(NULL); wait(NULL);
    return 0;
}
```

That's `ls / | grep e`, in pure C. Read it slowly. Run it. Marvel that you understand it.

## Exercise 5 — myls

Write `myls.c` that uses `stat()` (or `opendir`/`readdir` + `stat`) to list a directory like `ls -l`. Include permissions (decode the mode), size, and name.

Useful: `man 2 stat`, `man 3 opendir`, `man 3 readdir`.

## Reflection

Now look at `bash -x` or strace a real `ls > file.txt`. You'll see the exact pattern you just wrote: fork → open → dup2 → exec → wait. The shell is not magic — you can build one.
