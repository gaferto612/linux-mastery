# Module 09 — Solutions

## Exercise 1 — Inspect your network

```bash
ip addr            # interfaces + IPs
ip route           # routing table; "default via X dev Y" is your gateway
ip link            # link state (UP/DOWN, MAC)
ss -tulpn          # -t TCP, -u UDP, -l listening, -p process, -n no DNS resolution
hostname -I        # space-separated IPs (no IPv6 link-locals)
```

Reading `ip addr`:

```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
    link/ether 08:00:27:aa:bb:cc                    ← MAC
    inet 192.168.1.50/24 brd 192.168.1.255          ← IPv4 + netmask
    inet6 fe80::a00:27ff:feaa:bbcc/64               ← link-local IPv6
```

- The "real" interface is whichever shows `UP` and has a non-loopback address.
- `lo` (127.0.0.1) is always there — it's the loopback, software-only.
- Default gateway = the `default` route in `ip route`.

`ss -tulpn` shows what's listening; if SSH is up you'll see `*:22` or `0.0.0.0:22`.

---

## Exercise 2 — DNS three ways

```bash
dig anthropic.com              # most info (TTL, authoritative server, sections)
nslookup anthropic.com         # short summary
getent hosts anthropic.com     # uses /etc/nsswitch.conf — same lookup your apps use
```

`getent` is the most "real-world" lookup because it follows your system's full resolution path: `/etc/hosts` first, then DNS via `/etc/resolv.conf`. `dig` and `nslookup` go straight to DNS.

`/etc/resolv.conf` lists the resolvers. On modern systemd systems it points to `127.0.0.53` (systemd-resolved), which then forwards upstream.

After adding `127.0.0.1 myfakedomain.local` to `/etc/hosts`:

```bash
getent hosts myfakedomain.local   # works
dig myfakedomain.local            # NXDOMAIN — dig skipped /etc/hosts
```

That's the difference between "DNS" and "name resolution".

---

## Exercise 3 — ping and traceroute

```bash
ping -c 4 8.8.8.8
ping -c 4 google.com
traceroute google.com           # 30 hops max; each hop = one router
```

**Diagnostic table:**

| ping 8.8.8.8 | ping google.com | what's broken                              |
|-------------|------------------|--------------------------------------------|
| ✅ works    | ✅ works         | nothing                                    |
| ✅ works    | ❌ fails         | DNS — IP reachable, name not resolving     |
| ❌ fails    | ❌ fails         | no internet at all (gateway? route? cable?)|
| ❌ fails    | ✅ works         | weird — caching? `/etc/hosts` entry?       |

Traceroute uses TTL tricks: it sends packets with TTL=1, 2, 3 … each router decrements TTL and when it hits 0 the router sends back ICMP "time exceeded" — that's how you learn the hop.

---

## Exercise 4 — SSH keys

```bash
ssh-keygen -t ed25519 -C "me@example.com"
# files created:
#   ~/.ssh/id_ed25519       ← PRIVATE (never share, mode 600)
#   ~/.ssh/id_ed25519.pub   ← public  (paste anywhere, harmless)
```

Why ed25519: smaller, faster, more secure than RSA. Use it unless you're connecting to ancient hardware that only knows RSA.

```bash
ssh-copy-id user@server     # appends your pubkey to server:~/.ssh/authorized_keys
ssh user@server             # works without password
```

What `ssh-copy-id` does under the hood:

```bash
cat ~/.ssh/id_ed25519.pub | ssh user@server "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

The strict permissions matter — sshd will reject your key if `.ssh` is group-writable.

---

## Exercise 5 — ~/.ssh/config

```
Host myvm
    HostName 192.168.1.50
    User myuser
    IdentityFile ~/.ssh/id_ed25519
    Port 22
    ServerAliveInterval 60

Host *.internal
    User devops
    ProxyJump bastion.example.com
```

`Host` patterns are matched top-down; multiple matches all apply (later doesn't override). `ProxyJump bastion` is the modern, clean replacement for `-o ProxyCommand`.

---

## Exercise 6 — ufw

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh          # = allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw enable             # SSH must already be allowed first!
sudo ufw status verbose
```

> 🔥 **Order matters.** Always `allow ssh` BEFORE `enable` on a remote host. Otherwise you `enable`, the default-deny kicks in, your SSH session dies, and you're locked out.

Under the hood ufw writes iptables/nftables rules. `sudo iptables -L -n -v` shows the actual chains.

---

## Exercise 7 — Port forwarding

```bash
ssh -L 8080:localhost:5432 user@server
```

What this means:

```
   YOUR machine                          SERVER
   ───────────────                       ───────────────
   localhost:8080  ──── SSH tunnel ────▶ server's localhost:5432
        ▲                                       │
        │                                       ▼
   psql -h localhost -p 8080                postgres
```

The `localhost` in `localhost:5432` is from the **server's** point of view — what address the server should connect to once the SSH tunnel terminates.

Other flavors:
- `-L` local forward (above)
- `-R` remote forward — expose YOUR port on the server
- `-D` dynamic / SOCKS proxy — turn SSH into a poor-man's VPN

---

## Exercise 8 — Troubleshooting workflow

The canonical layered debug for "I can't reach this server":

```
 1.  Is the process running?            systemctl status / ps
 2.  Is it listening on the right port? ss -tlnp | grep :80
 3.  Local connect works?               curl http://localhost:80
 4.  External connect works?            curl http://<server-ip>:80 from another box
 5.  Firewall?                          sudo ufw status / iptables -L
 6.  DNS?                               dig the.name.you.expect
 7.  TLS valid?                         curl -v https://… (cert errors show here)
```

Always start at the lowest layer that could be broken. Don't suspect TLS before you've confirmed the process is even running.

---

## "It's always DNS"

Because:
- DNS is cached at many layers (resolver, application, browser, OS).
- DNS records can be wrong AND look right (mismatch between dev box and production).
- TTLs delay propagation of fixes.
- New TLDs, glue records, DNSSEC, search domains — many ways to be subtly wrong.
- Symptoms look like everything else: "connection refused", "no route", "timeout".

When something is broken and you've ruled out everything else, check DNS again with `dig`, `getent hosts`, and from a different network.

---

## Common pitfalls

- **`ufw enable` over SSH without first allowing 22** → instant lock-out.
- **Private keys with wrong permissions** (mode 644) → ssh refuses to use them. `chmod 600`.
- **Mixing IPv4/IPv6** → "ping works but curl doesn't" because curl prefers IPv6 and your IPv6 path is broken.
- **Stale ARP / DNS caches** when things have changed. `sudo systemctl restart systemd-resolved`, or `ip neigh flush all`.
- **NAT confusion** — your VM's "external" IP might be the host's, not its own.

→ [Module 10](../../module-10-storage-and-filesystems/README.md)
