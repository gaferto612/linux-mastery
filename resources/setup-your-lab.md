# Setting Up Your Linux Lab

You need a Linux system you can break, fix, and break again. Pick the option that fits your situation.

---

## Option 1 — Local Virtual Machine (recommended)

This is the best learning environment. Everything is isolated from your real OS, and snapshots let you roll back when you mess things up (which you will, intentionally).

### What you need

- A computer with 8 GB RAM minimum (16 GB ideal), 30 GB free disk space
- Virtualization software (free):
  - **Windows / Linux:** [VirtualBox](https://www.virtualbox.org/)
  - **macOS Intel:** VirtualBox
  - **macOS Apple Silicon (M1/M2/M3):** [UTM](https://mac.getutm.app/)
- An Ubuntu ISO: [https://ubuntu.com/download/desktop](https://ubuntu.com/download/desktop) (LTS version, latest)

### Setup steps (VirtualBox)

1. Install VirtualBox.
2. Download Ubuntu Desktop LTS (~5 GB).
3. In VirtualBox: New → Name "linux-lab" → Type Linux → Version Ubuntu (64-bit).
4. Memory: 4 GB minimum.
5. Create a virtual hard disk: 30 GB, VDI, dynamically allocated.
6. Settings → Storage → click the empty CD → choose your Ubuntu ISO.
7. Start the VM. Install Ubuntu (defaults are fine).
8. After install: **take a snapshot** ("clean install"). When you break things badly, restore to this.

### Useful settings

- Devices → Insert Guest Additions CD → install — gets you copy-paste between host and VM, better display.
- Network: NAT is fine for general use. For pentest module (19), you'll add a second host-only adapter.

---

## Option 2 — WSL2 on Windows

If you already have Windows and don't want to deal with VMs.

```powershell
wsl --install -d Ubuntu
```

Reboot, set username + password, you're in. Limitations:
- No graphical install
- Some system-level stuff (systemd, networking quirks) is different from "real" Linux
- For Module 19 (pentesting), you'll still need actual VMs

WSL2 is fine for Modules 1–14 and 15–17. For the rest, lean toward Option 1 or 3.

---

## Option 3 — Cheap VPS

Rent a small Linux server for $4–6/month. Bonus: you also practice working over SSH, which is most real-world Linux work.

Recommended providers (no affiliation):
- **Hetzner** (Germany — cheap, EU privacy)
- **DigitalOcean**
- **Vultr**
- **Linode (Akamai)**

Get the smallest "droplet" / instance running Ubuntu LTS. Connect over SSH:

```bash
ssh root@<your-server-ip>
```

Limitations:
- You can't snapshot freely (some providers charge)
- Real internet = real attackers will hit you immediately. Configure ufw and SSH-key-only auth before doing anything else.
- For Module 19 (pentesting), **never use this** — only attack your own isolated lab VMs, never real internet hosts.

---

## What about Mac/Windows directly?

You can install Linux tools natively, but you'll miss the experience of actually using Linux. Stick with one of the options above.

---

## After setup — first commands

Whichever option you picked, log in and run:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install git curl wget vim tmux htop tree
```

These are tools you'll use throughout the course. Clone this course repo:

```bash
cd ~
git clone https://github.com/<yourusername>/linux-mastery.git
cd linux-mastery
```

(After you push it to GitHub — see [`github-setup.md`](github-setup.md).)

Now go to [Module 01](../module-01-terminal-basics/README.md) and get started.
