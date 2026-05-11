# Networking Cheatsheet

## Interface info
| Command | What it does |
|---|---|
| `ip addr` | All interfaces and their IPs |
| `ip addr show eth0` | Just one interface |
| `ip link` | Link-level info |
| `ip -s link` | With packet/byte counts |
| `hostname` | Machine's hostname |
| `hostname -I` | My IP addresses |

## Routing
| Command | What it does |
|---|---|
| `ip route` | Routing table |
| `ip route get 8.8.8.8` | Which route reaches this IP? |
| `ip route add 10.0.0.0/24 via 192.168.1.1` | Add a route |

## Connections and ports
| Command | What it does |
|---|---|
| `ss -tulpn` | TCP+UDP listening, with processes, numeric |
| `ss -tan` | All TCP, numeric |
| `ss -tan state established` | Just established |
| `ss -tn dst :443` | Connections to port 443 |
| `lsof -i :80` | Who has port 80 open |
| `netstat -tulpn` | Older equivalent of ss |

## DNS
| Command | What it does |
|---|---|
| `dig example.com` | Lookup |
| `dig +short example.com` | Just the answer |
| `dig @8.8.8.8 example.com` | Ask a specific server |
| `dig example.com MX` | MX records |
| `dig -x 1.2.3.4` | Reverse lookup |
| `nslookup example.com` | Simpler/older lookup |
| `getent hosts example.com` | Same as the system's resolver |
| `host example.com` | Quick lookup |

## Reachability
| Command | What it does |
|---|---|
| `ping -c 4 host` | 4 pings |
| `ping6 -c 4 host` | IPv6 |
| `traceroute host` | Path to host |
| `mtr host` | Continuous traceroute (great for diagnosing) |
| `nc -zv host 22` | Is port 22 open? |
| `nc -l 1234` | Listen on port 1234 |

## curl / wget
```bash
curl example.com                      # GET, print to stdout
curl -O example.com/file.zip          # download, keep filename
curl -L example.com                   # follow redirects
curl -v example.com                   # verbose (great for debugging)
curl -I example.com                   # HEAD request, just headers
curl -X POST -d 'a=1&b=2' example.com # POST form data
curl -H 'Authorization: Bearer xyz' example.com   # custom header
wget example.com/file.zip             # alternative downloader
```

## SSH
```bash
ssh user@host                         # basic
ssh -p 2222 user@host                 # custom port
ssh -i ~/.ssh/key user@host           # specific key
ssh -L 8080:localhost:80 user@host    # local port forward
ssh -R 8080:localhost:80 user@host    # reverse port forward
ssh -D 1080 user@host                 # SOCKS proxy
ssh -J jumphost user@target           # via jump host
ssh-copy-id user@host                 # install your pubkey
scp file user@host:/path              # copy to remote
scp user@host:/path file              # copy from remote
rsync -av file user@host:/path        # better than scp for most things
```

## ~/.ssh/config example
```
Host myvm
    HostName 192.168.1.50
    User myuser
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
```

## ufw quick reference
```bash
sudo ufw status verbose
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow from 10.0.0.0/8         # from a subnet
sudo ufw delete allow 80/tcp
sudo ufw enable
sudo ufw disable
```

## Troubleshooting order (when network breaks)

1. Interface up and has IP? `ip addr`
2. Default route exists? `ip route`
3. Can ping gateway? `ping <gateway>`
4. Can ping internet IP? `ping 1.1.1.1`
5. DNS works? `dig example.com`
6. Can curl a site? `curl -v https://example.com`

If you get a "yes" then a "no", the layer between is broken.
