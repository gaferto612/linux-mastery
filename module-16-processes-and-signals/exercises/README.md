# Module 16 — Exercises

## Exercise 1 — fork

`fork1.c`:

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main(void) {
    printf("Before fork, PID = %d\n", getpid());

    pid_t pid = fork();
    if (pid < 0) {
        perror("fork"); return 1;
    } else if (pid == 0) {
        printf("Child: PID = %d, parent = %d\n", getpid(), getppid());
    } else {
        printf("Parent: PID = %d, child = %d\n", getpid(), pid);
        wait(NULL);   // wait for child to exit
    }
    return 0;
}
```

Compile and run several times. Sometimes the parent prints first, sometimes the child. Why?

## Exercise 2 — fork + exec

`runcmd.c`:

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main(void) {
    pid_t pid = fork();
    if (pid == 0) {
        execlp("ls", "ls", "-la", "/", (char *)NULL);
        perror("exec");      // only reached if exec failed
        return 1;
    }
    int status;
    waitpid(pid, &status, 0);
    printf("Child exited with %d\n", WEXITSTATUS(status));
    return 0;
}
```

This is the heart of every shell command. Strace it to see fork/execve/wait4 in action.

## Exercise 3 — Tiny shell

`tinyshell.c`:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/wait.h>

int main(void) {
    char line[1024];
    while (1) {
        printf("$ ");
        fflush(stdout);                       // important: line without \n won't flush by default
        if (!fgets(line, sizeof(line), stdin)) break;
        line[strcspn(line, "\n")] = 0;        // strip trailing newline
        if (strlen(line) == 0) continue;
        if (strcmp(line, "exit") == 0) break;

        pid_t pid = fork();
        if (pid < 0) { perror("fork"); continue; }
        if (pid == 0) {
            execlp(line, line, (char *)NULL);
            perror("exec");                    // only reached if exec failed
            exit(1);
        }
        wait(NULL);
    }
    return 0;
}
```

Try: `ls`, `pwd`, `date`, `exit`.

Extend it: parse arguments (split on spaces) so you can run `ls -l`.

## Exercise 4 — Signal handler

`catchsig.c`:

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

void handler(int sig) {
    const char *msg = "Caught SIGINT, ignoring...\n";
    write(1, msg, 28);   // write() is async-signal-safe; printf is not
}

int main(void) {
    signal(SIGINT, handler);
    while (1) {
        printf("running...\n");
        sleep(1);
    }
}
```

Run it. Press Ctrl-C — it doesn't die. Use Ctrl-\ (SIGQUIT) or `kill -9` from another terminal to stop it.

## Exercise 5 — Make a zombie

`zombie.c`:

```c
#include <stdio.h>
#include <unistd.h>

int main(void) {
    pid_t pid = fork();
    if (pid == 0) {
        printf("Child %d exiting\n", getpid());
        return 0;
    }
    printf("Parent %d sleeping. Look at ps now!\n", getpid());
    sleep(60);
    return 0;
}
```

While it sleeps, in another terminal:

```bash
ps -ef | grep defunct          # zombie processes show as <defunct>
```

The fix is to call `wait()` — add it before the sleep and re-run.

## Reflection

Why does Unix separate fork() and exec() into two calls instead of one? Find out about "redirection" — what does the shell do between fork() and exec() that wouldn't be possible if they were combined?
