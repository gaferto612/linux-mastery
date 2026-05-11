# Module 09 — Exercises

## Exercise 1 — Inspect your network

```bash
ip addr                    # interfaces and their IPs
ip route                   # routing table
ip link                    # link-level info
ss -tulpn                  # listening ports (TCP, UDP, listening, processes, numeric)
hostname -I                # your IP addresses
```

In your notes:
- Which interface is your "real" one?
- What's your default gateway?
- What's listening on port 22 (if anything)?

## Exercise 2 — DNS three ways

```bash
dig anthropic.com
nslookup anthropic.com
getent hosts anthropic.com
```

Do they agree? Look at `/etc/resolv.conf` — what server is doing the lookup?

Add a line to `/etc/hosts`: `127.0.0.1 myfakedomain.local`. Now `ping myfakedomain.local`. It works! That's `/etc/hosts` overriding DNS.

## Exercise 3 — ping and traceroute

```bash
ping -c 4 8.8.8.8
ping -c 4 google.com
traceroute google.com      # may need: sudo apt install traceroute
```

If ping to 8.8.8.8 works but ping to google.com doesn't, what's broken? (DNS.)

## Exercise 4 — SSH keys

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
# Press enter for default location, optional passphrase
cat ~/.ssh/id_ed25519.pub   # this is your public key — share this
cat ~/.ssh/id_ed25519       # this is your private key — NEVER share
```

Copy your public key to a server (or your VM if it has SSH):

```bash
ssh-copy-id user@server
ssh user@server             # should not ask for password now
```

## Exercise 5 — ~/.ssh/config

Create `~/.ssh/config`:

```
Host myvm
    HostName 192.168.1.50
    User myuser
    IdentityFile ~/.ssh/id_ed25519
```

Now `ssh myvm` works without all the typing.

## Exercise 6 — ufw

```bash
sudo ufw status
sudo ufw allow ssh
sudo ufw allow 80
sudo ufw enable
sudo ufw status verbose
```

**Important:** Before you `ufw enable`, make sure SSH is allowed if you're on a remote machine — otherwise you'll lock yourself out!

## Exercise 7 — Port forwarding

If you have a service on a remote machine that only listens on `127.0.0.1` (e.g., a database):

```bash
ssh -L 8080:localhost:5432 user@server
```

Now connecting to `localhost:8080` locally goes through SSH to port 5432 on the remote. This is how you securely access internal services.

## Exercise 8 — Troubleshooting workflow

Pretend a friend says: "my web server isn't working". Walk through:

1. Is the process running? (`ps`, `systemctl status`)
2. Is it listening on the right port? (`ss -tulpn`)
3. Can you connect locally? (`curl localhost:80`)
4. Can you connect from outside? (`curl <ip>`)
5. Firewall? (`ufw status`)
6. DNS? (`dig`)

Write this checklist in your notes. You'll use it constantly.

## Reflection

Why is "it's always DNS" a famous sysadmin joke?
