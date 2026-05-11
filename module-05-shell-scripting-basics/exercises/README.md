# Module 05 — Exercises

## Exercise 1 — Your first script

Create `hello.sh`:

```bash
#!/bin/bash
echo "Hello, world!"
```

Make it executable: `chmod +x hello.sh`. Run it with `./hello.sh`.

Now extend it: ask the user their name, then greet them.

## Exercise 2 — Variables and quoting

Write a script that demonstrates the difference between:

```bash
greeting="hello world"
echo $greeting       # what gets printed?
echo "$greeting"
echo '$greeting'
echo "$greeting     extra spaces"
echo $greeting     extra spaces
```

In your notes: explain in your own words why these differ.

## Exercise 3 — A backup script

Write `backup.sh` that:
1. Takes one argument: a directory to back up.
2. Creates a copy of it with a timestamp in the name: `mydir-2026-05-12-14-30.tar.gz`
3. Prints "Backup complete: <filename>".
4. If no argument is provided, prints "Usage: ./backup.sh <directory>" and exits with code 1.

Useful: `date +%Y-%m-%d-%H-%M`, `tar -czf`.

## Exercise 4 — Conditional logic

Write `is-image.sh` that:
1. Takes a filename as argument.
2. Checks if the file ends in `.jpg`, `.png`, `.gif`, or `.webp`.
3. Prints "Image" or "Not an image".

Use a `case` statement (research this — it's cleaner than nested `if`).

## Exercise 5 — Loops

Write `count-lines.sh` that:
1. Loops over every `.txt` file in the current directory.
2. For each file, prints `<filename>: <number of lines>`.

Hint: `wc -l` counts lines.

## Exercise 6 — Arguments and validation

Write `greet.sh` that:
1. Takes 1 to 3 names as arguments.
2. Greets each one: "Hello, <name>!"
3. If no names given, exits with an error.
4. If more than 3, prints "Too many names" and exits with code 2.

Use `$#`, `$@`, and conditionals.

## Exercise 7 — Exit codes

Write a script `check.sh` that:
1. Takes a filename as argument.
2. If the file exists and is readable, exit with 0 (and print "OK").
3. If the file doesn't exist, exit with 1 (print "Not found").
4. If it exists but isn't readable, exit with 2 (print "Not readable").

Test by running:

```bash
./check.sh /etc/hostname; echo "Exit: $?"
./check.sh /nope; echo "Exit: $?"
./check.sh /etc/shadow; echo "Exit: $?"   # likely not readable
```

## Exercise 8 — The shellcheck habit

Install `shellcheck` (`sudo apt install shellcheck`) and run it on all your scripts above:

```bash
shellcheck backup.sh
```

It will flag bugs and bad habits. **Get used to running it.** It's the #1 way to learn solid bash.

## Reflection

What pattern keeps coming back in these scripts? (Hint: check args → validate → do the work → exit cleanly.)
