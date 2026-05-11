# Solutions

Solutions for this module are intentionally light — you grow more by working through the exercises and checking your own answers against the man pages and book chapters.

## How to verify your own work

1. **Re-read the relevant book section.** If your output matches the example, you're on track.
2. **Use `man <command>`** to confirm a flag does what you think it does.
3. **Run `shellcheck`** on every script you write (Module 5+).
4. **Compare to "real" Linux behavior** — if you wrote a `mycat`, diff its output against `cat`.

## Common pitfalls to self-check

- Did you handle the case where a command might fail?
- Did you quote your variables (`"$var"` not `$var`)?
- Did you check exit codes when it matters?
- Does your script work when run from a different directory?

## Stuck for more than 30 minutes?

Open a GitHub issue using the "Stuck or confused" template. Writing it out often unsticks you. If not, ask an LLM, Stack Overflow, or the Unix StackExchange — but write your understanding *first*, then ask.

## Want a worked solution?

Add an issue requesting it and link the specific exercise. As this repo evolves, I (you) will fill in solutions for the trickier ones.
