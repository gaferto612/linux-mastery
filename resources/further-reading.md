# Resources & Further Reading

Beyond the five course books, these are battle-tested resources for going deeper or getting unstuck.

## Online references

- **`man` pages** — your first stop, always. `man <command>` or `man <section> <topic>`.
- **`tldr` pages** — friendlier than man. Install: `sudo apt install tldr`. Try `tldr tar`.
- **[explainshell.com](https://explainshell.com)** — paste a command, get every flag explained.
- **[shellcheck.net](https://www.shellcheck.net)** — paste a script, get warnings. Indispensable.
- **[regex101.com](https://regex101.com)** — interactive regex builder/tester.

## Documentation gold

- **[Arch Wiki](https://wiki.archlinux.org)** — the best Linux documentation, even if you're not on Arch.
- **[Debian Administrator's Handbook](https://debian-handbook.info/browse/stable/)** — free, Debian-focused.
- **[Linux Documentation Project](https://tldp.org)** — older but classics.
- **[The Linux Kernel docs](https://www.kernel.org/doc/html/latest/)** — when you go deep.

## Communities

- **[r/linuxquestions](https://reddit.com/r/linuxquestions)** — friendly to beginners.
- **[r/linux4noobs](https://reddit.com/r/linux4noobs)** — even more so.
- **[Server Fault](https://serverfault.com)** — Q&A for sysadmins. Search first; questions answered well are usually old.
- **[Unix & Linux Stack Exchange](https://unix.stackexchange.com)** — best for "how does this work" questions.

## Security-specific

- **[HackTheBox](https://hackthebox.com)** — gamified pentest practice.
- **[TryHackMe](https://tryhackme.com)** — guided rooms, more beginner-friendly than HTB.
- **[OverTheWire Wargames](https://overthewire.org/wargames/)** — free, CLI-based, classic.
- **[OWASP](https://owasp.org)** — web security canon.
- **[CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/)** — gold-standard hardening checklists.

## Books beyond your five

When you outgrow this course:

- **"UNIX Network Programming" — W. Richard Stevens** — the bible for socket programming.
- **"Advanced Programming in the UNIX Environment" — W. Richard Stevens** — companion to TLPI, slightly older but legendary.
- **"Site Reliability Engineering" — Google (free online)** — modern ops at scale.
- **"Designing Data-Intensive Applications" — Martin Kleppmann** — for when you start running real systems.
- **"The Web Application Hacker's Handbook"** — web pentesting depth.

## Newsletters & blogs worth subscribing to

- **[Linux Weekly News (LWN)](https://lwn.net)** — kernel and distro news, deeply technical.
- **[Phoronix](https://phoronix.com)** — Linux performance and gaming.
- **Julia Evans's blog** (jvns.ca) — best explanations of Linux internals on the internet, with cartoons.

## Tools worth installing now

```bash
sudo apt install tldr tree htop ncdu jq ripgrep fd-find bat tmux
```

- `tree` — visual directory tree
- `htop` — better top
- `ncdu` — interactive disk usage
- `jq` — JSON processor (you'll use this constantly with APIs)
- `ripgrep` (`rg`) — grep but 10x faster and friendlier
- `fd-find` (`fd`) — find but friendlier
- `bat` — cat with syntax highlighting
- `tmux` — terminal multiplexer; keeps sessions alive on remote hosts
