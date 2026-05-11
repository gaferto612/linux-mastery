# Module 03 — Exercises

## Exercise 1 — Who's running what?

1. `ps` — what do you see? Only your shell?
2. `ps -ef` — now what?
3. `ps aux` — different format. Note CPU and memory columns.
4. `ps aux | grep bash` — find your shell process. Note the PID.

## Exercise 2 — Job control dance

```bash
sleep 300            # starts and blocks the terminal
```

Press **Ctrl-Z**. What does the shell say?

```bash
jobs                 # list jobs
bg                   # resume in background
jobs                 # now it should say "Running"
fg                   # bring back to foreground
```

Press **Ctrl-C** to kill it.

Repeat, but this time start it in the background directly: `sleep 300 &`.

## Exercise 3 — Sending signals

Start `sleep 1000 &`. Note the PID it prints.

```bash
kill <PID>           # default = SIGTERM (15). Polite request to exit.
```

Did it exit? Now do it again, but this time:

```bash
sleep 1000 &
kill -9 <PID>        # SIGKILL. Cannot be ignored. The nuclear option.
```

Read `man 7 signal` and list 5 signals with their names and meanings in your notes.

## Exercise 4 — top and htop

Install `htop` if you don't have it (`sudo apt install htop` on Debian/Ubuntu).

Run `top`. Press:
- `M` to sort by memory
- `P` to sort by CPU
- `q` to quit

Run `htop`. Notice the difference — colors, scrolling, mouse support. Press F1 to see help.

## Exercise 5 — The process tree

`pstree` — see the entire tree of processes. Note that everything goes back to one root (PID 1, usually systemd).

`pstree -p` — same, with PIDs.

Find your shell in the tree. Trace it up to PID 1.

## Exercise 6 — pgrep and pkill

```bash
pgrep bash           # find PIDs of any bash process
pgrep -a bash        # also show command line
pkill -f "sleep 300" # kill processes whose command matches "sleep 300"
```

Start a couple of sleeps and practice killing them by pattern instead of PID.

## Exercise 7 — Surviving disconnect

```bash
nohup sleep 600 &
exit                 # close your terminal
```

Open a new terminal. Is the sleep still running? (`ps -ef | grep sleep`)

This is how you start a process that lives beyond your SSH session. (Modern alternative: `tmux` or `screen` — try them later.)

## Exercise 8 — A process that won't die

Run this (it ignores SIGTERM):

```bash
trap '' TERM        # only works if you do this in a subshell or script
sleep 600 &
```

(On second thought: easier — find a system process you can see but can't kill, and try `kill` on it. You'll get "Operation not permitted." Why?)

## Reflection

Why does Linux let you tell `kill` to send any signal, but `kill -9` works "the same" most of the time? What's the *real* difference between SIGTERM and SIGKILL from the program's perspective?
