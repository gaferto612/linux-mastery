# Vim Survival Guide

You don't need to master vim. You need enough to edit a config file when you SSH into a server that doesn't have nano.

## The absolute minimum

| Key | What it does |
|---|---|
| `i` | Enter insert mode (start typing) |
| `Esc` | Leave insert mode (back to command mode) |
| `:w` | Save |
| `:q` | Quit |
| `:wq` | Save and quit |
| `:q!` | Quit without saving (discard changes) |

That's it. With those 6 commands, you can survive any vim encounter.

## Slightly more useful

| Key | What it does |
|---|---|
| `h j k l` | Left, down, up, right |
| `w` | Jump to next word |
| `b` | Jump back a word |
| `0` | Start of line |
| `$` | End of line |
| `gg` | Top of file |
| `G` | Bottom of file |
| `:N` | Go to line N |
| `dd` | Delete current line |
| `yy` | Copy current line (yank) |
| `p` | Paste after cursor |
| `u` | Undo |
| `Ctrl-R` | Redo |
| `/text` | Search forward for "text" (n / N for next / prev) |

## Made a mess? Just escape

If you're confused or stuck:
1. Press `Esc` a few times.
2. Type `:q!` and press Enter.
3. You're out with no changes saved. Start over with `vim file`.

## Modes — the one concept you must understand

vim has *modes*. This is what makes it weird at first.

- **Normal mode** (default) — keys are commands, not text.
- **Insert mode** — keys type text (this is "normal" in other editors).
- **Visual mode** — keys select text.
- **Command mode** — `:` then a command (`:w`, `:q`, etc).

Always: `Esc` brings you back to Normal mode. When in doubt, hit `Esc`.

## Don't want vim?

```bash
sudo apt install nano
export EDITOR=nano               # add to .bashrc to make permanent
```

`nano` is friendlier. Most servers have it. Use it without shame.

But: vim is everywhere, including bare-bones rescue systems. Knowing those 6 commands has saved many a sysadmin.

---

[← Back to course home](../README.md)
