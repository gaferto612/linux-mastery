# Module 03 — Solutions / Key Answers

## Signals you should know

| Signal | Number | Meaning | Catchable? |
|---|---|---|---|
| SIGHUP | 1 | "Hangup" — historically when modem disconnected; now often used to reload config | Yes |
| SIGINT | 2 | "Interrupt" — what Ctrl-C sends | Yes |
| SIGKILL | 9 | "Die now" — kernel kills the process; program cannot intercept | **No** |
| SIGTERM | 15 | "Please exit" — graceful termination request | Yes |
| SIGSTOP | 19 | "Pause" — process freezes | No |
| SIGCONT | 18 | "Continue" — resume a stopped process | Yes |

## Exercise 3 — SIGTERM vs SIGKILL

- SIGTERM: the program receives it, gets a chance to **clean up** (close files, save state, finish writes), then exit. Some programs ignore it (rare; usually it works).
- SIGKILL: the **kernel** terminates the process. The program never knows. No cleanup. Possible data loss.

**Rule:** Always try SIGTERM first. Use SIGKILL only when SIGTERM doesn't work.

## Exercise 7 — nohup

`nohup` does two things:
1. Makes the process ignore SIGHUP (which is sent to processes when the controlling terminal closes).
2. Redirects stdout/stderr to `nohup.out` so the process doesn't crash trying to write to a closed terminal.

Modern alternative: `disown` after `&`, or just use `systemd-run --user` or `tmux`.

## Exercise 8 — Can't kill it

`Operation not permitted` means you don't own the process. Only root can kill processes owned by other users (or root). This is a security boundary — otherwise any user could crash the system by killing essential services.

## "kill -9 always works" — but...

It works on processes you own. It works for root on most processes. It does **not** work on:
- Process in uninterruptible sleep (state `D` in `ps`) — usually stuck on I/O
- Kernel threads
- PID 1 (the init process)

If `kill -9` doesn't work, the process is probably stuck in the kernel and you may need to reboot.
