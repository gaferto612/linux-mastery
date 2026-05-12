# Module 20 — Capstone Projects

**Phase:** Synthesis · **Time:** 4–8 weeks · **Prereq:** Modules 1–19

You've reached the end. Now you build.

A capstone is **proof of skill** — to yourself, to future employers, to your future self when you forget what you knew. Pick at least one. Ideally two from different areas.

---

**In this module:** [📋 Projects](projects/README.md) · [📝 Notes](notes/my-notes.md)

## 🛤️ Pick your path

```
   🎓 You finished modules 1-19
              │
              ▼
   Where do you want to go?
              │
     ┌────────┼────────────────────────┐
     ▼        ▼                        ▼
   Path A · SysAdmin / DevOps   Path B · Systems Programming   Path C · Security
     │                            │                                │
     ├──▶ Self-hosted web service ├──▶ Mini shell in C             ├──▶ Harden + audit your VPS
     ├──▶ Ansible IaC starter     ├──▶ File-watcher daemon         ├──▶ Pentest your own lab
     └──▶ Prometheus + Grafana    └──▶ Custom systemd service      │      + write the report
          monitoring dashboard                                     └──▶ Build a honeypot
                                          │
                                          ▼
                                  📦 ship to GitHub
```

## 🏁 Definition of "done"

```
   [ ] runs from a clean install / clean VM
   [ ] README a stranger can follow end-to-end
   [ ] reproducible: scripts / playbooks / Dockerfile, not "I did stuff"
   [ ] failure modes documented (what breaks, how to recover)
   [ ] you can demo it on a call without hand-waving
```

---

## Why capstones matter

The modules taught you skills in isolation. Real Linux work is integrated: a problem requires shell scripting *and* networking *and* permissions *and* systemd, all at once. Capstones force that integration.

Also: these go on your GitHub. Hiring managers look at GitHub. A "Linux course completion" is meaningless; **a working project** is signal.

---

## Choose your project(s)

### Path A — Sysadmin / DevOps

**1. Self-hosted web service**
Set up a VPS or VM running:
- nginx serving a static site
- HTTPS via Let's Encrypt
- ufw firewall
- fail2ban
- Automated backups to a second location
- A systemd service for an app (anything — a simple Flask app, a Node app, whatever)
- A status page that lives at `/status` and reports uptime, disk usage, etc.

Document everything in a README. Treat it like onboarding docs for a stranger.

**2. Infrastructure-as-Code starter**
Use Ansible (or shell scripts if Ansible feels too much) to provision a fresh Ubuntu server from blank to your hardened baseline. The same playbook should be re-runnable and idempotent.

**3. Monitoring dashboard**
Set up Prometheus + Grafana on a VM. Scrape metrics from node-exporter. Build one dashboard showing CPU, memory, disk, network, load. Bonus: alert on disk > 90%.

### Path B — Scripting / automation

**4. System health reporter**
A bash script that:
- Runs as a daily systemd timer
- Collects: disk usage, top 5 memory processes, failed services, recent errors from journal, uptime
- Emails you a report
- Logs to its own journal entry

**5. Backup system**
End-to-end backup tool using restic or borg:
- Configured via a single config file
- Daily incremental, weekly full
- Encrypted, deduplicated
- Off-site replication
- Pruning policy
- Restore-test script you can run anytime

**6. Personal dotfiles repo**
Your `.bashrc`, `.vimrc`, `.tmux.conf`, `.ssh/config` (template), etc., in a git repo with an install script. So when you set up a new machine, it takes one command.

### Path C — Systems programming

**7. Mini-shell**
Extend the tiny shell from Module 16 into a real one:
- Builtins: `cd`, `pwd`, `exit`, `export`
- Pipes (using your Module 17 pipe code)
- Input/output redirection (`<`, `>`, `>>`)
- Background jobs with `&`
- Signal handling (Ctrl-C kills foreground, doesn't kill shell)

**8. tiny-cat / tiny-grep / tiny-ls**
Reimplement core Unix utilities in C. Match the spec from the man pages (just the common flags). Compare your output with the real ones using `diff`.

### Path D — Security

**9. Hardened Linux server build**
Take a fresh Ubuntu, follow CIS Benchmarks or your own hardening checklist, document each step. Run lynis at the start and end — show the score difference.

**10. CTF write-ups**
Solve 5 boxes on HackTheBox or TryHackMe (start with "easy"). Write a full write-up for each — what you tried, what worked, what you learned. This is the most respected portfolio in security.

---

## Standards for your capstone

A capstone project is "done" when:

- ✅ It actually works, on a fresh machine, end-to-end
- ✅ The README explains *why*, not just *what* — what problem you solved, why you made the choices you did
- ✅ Someone else could follow your README and reproduce it
- ✅ The code is in a git repo with sensible commit history (not one giant "initial commit")
- ✅ You've used it / tested it / restored from it / run it long enough to find at least one bug and fix it

---

## Reflection at the end

When you're done with your capstone(s), write a final entry in `progress/tracker.md`:

- What surprised you?
- What did you struggle with that turned out to matter most?
- What do you want to learn next?

That's the close of this course. Linux is a deep well — you've drawn enough to know how the bucket works. From here, it's just more drawing.

Good luck. ✊

---

**Navigate:** [← Previous module](../module-19-intro-to-pentesting/README.md) · [🏠 Home](../README.md) · —
