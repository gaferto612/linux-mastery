# Putting this course on GitHub

You learn by *doing*, including learning git. Let's get this repo onto your GitHub.

## 1. Create a GitHub account (if you don't have one)

[github.com/signup](https://github.com/signup) — free.

## 2. Install git

```bash
sudo apt install git           # Debian/Ubuntu
git --version
```

## 3. Configure git (one-time)

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
git config --global init.defaultBranch main
```

## 4. Create a new repo on GitHub

- Go to github.com → New repository
- Name: `linux-mastery` (or whatever)
- **Private or Public** — your call. Public is fine; people will see your progress.
- Don't add README/license here — we already have them.
- Click Create.

## 5. Set up SSH keys for GitHub

(You learned this in Module 09. Reuse those keys.)

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the output. Go to GitHub → Settings → SSH and GPG keys → New SSH key. Paste.

Test:

```bash
ssh -T git@github.com
```

You should see a greeting from GitHub.

## 6. Push your local repo

In your linux-mastery folder:

```bash
git init
git add .
git commit -m "Initial commit: course skeleton"
git branch -M main
git remote add origin git@github.com:<yourusername>/linux-mastery.git
git push -u origin main
```

Done. Your course is on GitHub.

## 7. Daily workflow

After each module (or each work session):

```bash
git add .
git commit -m "Module 03: finished exercises 1-5"
git push
```

Watch your contribution graph fill in. It's surprisingly motivating.

## A few git tips for beginners

- `git status` — what's changed?
- `git diff` — what exactly changed?
- `git log --oneline` — your history
- `git checkout <file>` — discard local changes to a file
- `.gitignore` — list files git should ignore (e.g., `*.log`, `secrets.txt`)

If git confuses you (it will), [oh-shit-git.com](https://ohshitgit.com/) is the kindest reference on the internet.

---

[← Back to course home](../README.md)
