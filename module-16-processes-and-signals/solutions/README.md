# Module 16 — Solutions

## Exercise 1 — fork

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main(void) {
    printf("Before fork, PID = %d\n", getpid());

    pid_t pid = fork();
    if (pid < 0)      { perror("fork"); return 1; }
    else if (pid == 0){ printf("Child: PID=%d, parent=%d\n", getpid(), getppid()); }
    else              { printf("Parent: PID=%d, child=%d\n", getpid(), pid);
                        wait(NULL); }
    return 0;
}
```

`fork()` returns twice:

- In the parent: child's PID (positive integer).
- In the child: 0.
- On failure: -1 (only in the would-be parent — there's no child).

**Why output order varies:** the scheduler is free to run either process first. Both write to the same (line-buffered) stdout. If your terminal is a tty the buffer is flushed on `\n`, but order between the two processes is still up to the kernel.

> 🔥 **Footgun:** `printf` without `\n` before `fork()` puts text in the stdio buffer. After fork, BOTH processes own that buffered text → it gets printed twice. Always `fflush(stdout)` before forking, or use `write()` directly.

---

## Exercise 2 — fork + exec

```c
pid_t pid = fork();
if (pid == 0) {
    execlp("ls", "ls", "-la", "/", (char *)NULL);
    perror("exec");        // only reached if exec failed
    return 1;
}
int status;
waitpid(pid, &status, 0);
printf("Child exited with %d\n", WEXITSTATUS(status));
```

`exec` family naming convention (`man 3 exec`):

```
execl   list of args:  execl(path, arg0, arg1, …, NULL)
execv   array of args: execv(path, argv)
   p    PATH search:   execlp / execvp
   e    custom env:    execle / execve
```

`execve` is the actual syscall — everything else is a libc wrapper.

`WEXITSTATUS(status)` extracts the exit code (0–255) from the wait status. Other macros:

```c
WIFEXITED(status)   // exited normally?
WEXITSTATUS(status) // → 0..255
WIFSIGNALED(status) // killed by signal?
WTERMSIG(status)    // → signal number
WIFSTOPPED(status)  // stopped (Ctrl-Z)?
```

strace this to see fork → execve → wait4 in action:

```bash
strace -f ./runcmd 2>&1 | grep -E "fork|execve|wait"
```

---

## Exercise 3 — Tiny shell

The version in the exercise misses two big things any shell needs:

1. **Argument parsing.** `execlp(line, line, NULL)` treats the whole line as one argv[0].
2. **Built-ins.** `cd` can't be an external command — must be implemented in-process.

Better:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/wait.h>

#define MAX_ARGS 64

int main(void) {
    char line[1024];
    while (1) {
        printf("$ "); fflush(stdout);
        if (!fgets(line, sizeof line, stdin)) break;
        line[strcspn(line, "\n")] = 0;
        if (line[0] == 0) continue;

        // split on whitespace
        char *argv[MAX_ARGS];
        int argc = 0;
        char *tok = strtok(line, " \t");
        while (tok && argc < MAX_ARGS - 1) {
            argv[argc++] = tok;
            tok = strtok(NULL, " \t");
        }
        argv[argc] = NULL;

        // built-ins (must run in the shell process, not a child)
        if (strcmp(argv[0], "exit") == 0) break;
        if (strcmp(argv[0], "cd") == 0) {
            const char *target = argc > 1 ? argv[1] : getenv("HOME");
            if (chdir(target) < 0) perror("cd");
            continue;
        }

        pid_t pid = fork();
        if (pid < 0)  { perror("fork"); continue; }
        if (pid == 0) {
            execvp(argv[0], argv);
            perror(argv[0]);
            exit(127);          // command-not-found convention
        }
        int status;
        waitpid(pid, &status, 0);
    }
    return 0;
}
```

Why `cd` must be a built-in: `cd` changes the working directory of a process. If you fork and `cd` in the child, the child immediately exits — your shell's CWD is unchanged. Built-ins run in the shell itself.

Same logic for `exit`, `export`, `alias`, `set`, anything that mutates shell state.

---

## Exercise 4 — Signal handler

```c
void handler(int sig) {
    const char *msg = "Caught SIGINT, ignoring...\n";
    write(1, msg, 28);    // write() is async-signal-safe; printf is NOT
}

int main(void) {
    signal(SIGINT, handler);
    while (1) { printf("running...\n"); sleep(1); }
}
```

The async-signal-safe rule: a signal can arrive at *any* instruction — including in the middle of `malloc` holding a lock. If your handler calls `malloc` too, you can deadlock. Worse, `printf` uses a buffered FILE* that's almost certainly mid-mutation.

The whitelist of safe functions is in `man 7 signal-safety`. Memorize the practical ones: `write`, `_exit`, `signal`, `kill`, `raise`, simple sig-atomic flag sets.

Modern advice: use `sigaction()` instead of `signal()` — `signal()` semantics differ across systems.

```c
struct sigaction sa = { .sa_handler = handler, .sa_flags = SA_RESTART };
sigemptyset(&sa.sa_mask);
sigaction(SIGINT, &sa, NULL);
```

`SA_RESTART` makes interrupted syscalls auto-retry instead of returning EINTR.

---

## Exercise 5 — Zombies

```c
pid_t pid = fork();
if (pid == 0) {
    printf("Child %d exiting\n", getpid());
    return 0;            // exits immediately
}
printf("Parent %d sleeping. Look at ps now!\n", getpid());
sleep(60);               // forgot to wait()
return 0;
```

```bash
ps -ef | grep defunct
# you  12345  12344  0 14:00 pts/1  00:00:00 [zombie] <defunct>
```

A zombie occupies a slot in the kernel's process table holding the exit status. Until the parent calls `wait()`, that slot can't be reclaimed. Thousands of zombies → PID exhaustion.

Three ways to avoid zombies:

1. `wait()` / `waitpid()` after every fork.
2. Ignore SIGCHLD: `signal(SIGCHLD, SIG_IGN)` — kernel auto-reaps for you. Portable, easy.
3. Install a SIGCHLD handler that reaps in a loop:

```c
void reaper(int sig) {
    while (waitpid(-1, NULL, WNOHANG) > 0) {}
}
```

If the parent itself dies before reaping, the child gets reparented to PID 1 (systemd/init), which reaps it. So zombies only persist while the parent is alive and not waiting.

---

## Why fork/exec are two calls, not one

If they were combined (`spawn(prog, args)`), you couldn't do anything *between* "I have a child process" and "the new program is running". But that gap is precisely where the shell does its work:

```c
pid_t pid = fork();
if (pid == 0) {
    // I am the child. I'm still the shell — but with a copy of its state.

    // Redirect stdout to a file? Yes, before exec:
    int fd = open("out.txt", O_WRONLY|O_CREAT|O_TRUNC, 0644);
    dup2(fd, 1);  close(fd);

    // Drop privileges?  setuid(nobody);
    // Change directory? chdir("/var/lib/myapp");
    // Set env vars?     setenv("LANG", "C", 1);

    // NOW become the real program — inherits all of the above:
    execvp("cat", (char *[]){"cat", NULL});
}
```

`>`, `<`, `|`, `2>&1`, environment manipulation, working-directory changes — every one of these is shell code running in the child between fork and exec. That's why pipelines and redirection are language-agnostic: they're not features of `ls` or `grep`, they're features of fork/exec.

Windows `CreateProcess` bundles it all together → more API surface for fewer use cases.

---

## Common pitfalls

- **No `fflush(stdout)` before fork** → text printed twice.
- **`exec` succeeds → next line doesn't run.** People often write `execvp(…); printf("running");`. The printf only fires if exec FAILS.
- **`signal()` vs `sigaction()`** — `signal()` may reset the handler after each signal on some systems. Use `sigaction()`.
- **Calling `printf`/`malloc` in a signal handler** — deadlocks, weird output. Async-signal-safe only.
- **Forgetting to wait** → zombies pile up.
- **Race conditions in `waitpid(-1, …, WNOHANG)`** loops — handle EINTR, EAGAIN, ECHILD properly.

→ [Module 17](../../module-17-files-and-io/README.md)
