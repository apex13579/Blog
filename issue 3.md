# Home Lab Notes: Stabilizing the Stack

**Issue 03 — From Chaos to Clean Architecture**
**Date: May 2, 2026**
**Tags:** `Proxmox` `Docker` `Networking` `SQLite`

---

## Why I Do This on Old Hardware on Purpose

My home lab runs on cheap, used hardware. That is not a mistake or a budget problem — it is the whole point.

In the IT world, people love to say "just throw more RAM at it" when something runs slow. If you work at a company where downtime costs money, that makes sense. But for someone learning, buying expensive hardware actually gets in the way. When you only have 4 or 8GB of RAM to work with, you are forced to understand what is eating it. You learn why a service is slow. You figure out what you actually need versus what just sounds cool.

The goal is to prove you do not need a server rack or a big power bill to build something real. A $20 laptop from a thrift store can run a serious setup if you know what you are doing. And if you do not know yet — that is what the lab is for.

---

## The Foundation: Why I Use Proxmox

The most important early decision was picking **Proxmox VE** as my virtualization platform. Proxmox is a Type-1 hypervisor — it runs directly on the hardware with no Windows or macOS sitting underneath it.

> **Type-1 vs Type-2**
>
> Type-2 hypervisors (VirtualBox, VMware Workstation) run as apps inside another OS. Every VM call has to go through the host first — that is overhead you cannot reclaim.
>
> Type-1 (Proxmox) sits directly on bare metal. VMs and containers talk straight to the CPU and RAM. In a memory-constrained environment, that difference is felt immediately.

Inside Proxmox I run what I call a **Command Server** — one VM that handles everything management-related: monitoring, container oversight, and remote access. The logic is simple: if an experiment crashes something, I still need a way back in. Keeping the management layer separate and giving it resource priority means I always have that lifeline.

---

## 3D Printing My Way to a Better Setup

Professional cable management gear is expensive — sometimes more than the hardware it holds. So I started printing my own.

This is not just aesthetics. Bad cable routing in a small space traps hot air, causing hardware to overheat over time. I printed cable guides, a small patch panel, and mounting brackets for my mini PCs.

> **⚠ Material Note**
>
> Use **PETG or PLA+** for anything near sustained heat. Standard PLA deforms around 60°C — well within reach of a compact lab running 24/7. PETG handles it without issue.

Money saved on enclosures and mounts goes toward things that actually matter: faster SSDs, more RAM.

---

## The Day Everything Went Down

A few weeks ago, Vaultwarden, ntfy, and Portainer all stopped responding at the same time. First instinct: the containers crashed. They had not.

The real problem was at the host's network layer. Two programs — a stale Nginx install and a CUPS print service — had auto-started on boot and grabbed the ports my Docker containers needed. In Linux, only one process can bind to a port at a time. When Docker tried to set up its internal routing, port 80 was already taken. The containers were running fine. They just had no path to receive traffic.

> **🔴 Key Lesson**
>
> **"The container is running"** and **"the container is reachable"** are two completely different things. Container status and network reachability live on separate layers — they can fail independently of each other.

---

## How I Fixed It: The Nuclear Reset

Restarting the containers did nothing — which makes sense in hindsight. The containers were not the problem. The network routing underneath them was.

**Steps that actually worked:**

1. **Stop all containers** — halt everything before touching the network layer
2. **Find what is squatting on your ports** — use `ss -tulpn` or `lsof -i` to see what process holds each port
3. **Kill and permanently disable the offenders** — stop the process and prevent it auto-starting on next boot
4. **Restart the Docker daemon** — `systemctl restart docker` flushes stale iptables rules and rebuilds all port mappings and virtual bridges from scratch automatically
5. **Verify with curl** — a clean HTTP response confirms the proxy is up and routing correctly

```bash
# nuclear_reset.sh

# Step 1: halt all running containers
docker stop $(docker ps -aq)

# Step 2: find who is holding your ports
sudo ss -tulpn | grep -E ':80|:631'
# alternative: sudo lsof -i :80

# Step 3: disable the offenders permanently
sudo systemctl disable --now nginx
sudo systemctl disable --now cups

# Step 4: restart Docker
# This flushes iptables and rebuilds all NAT rules and bridges automatically
sudo systemctl restart docker

# Step 5: confirm the stack is reachable
curl -I http://localhost:80
```

The whole diagnosis and fix took about 45 minutes once I understood what was actually wrong. The containers were never the problem.

---

## IP Addresses: More Than Just Numbers

One thing the Google Cybersecurity course drilled into me early: you cannot secure a network you do not understand. An IP address is a 32-bit binary number that determines how traffic gets routed — not just a label on a device.

In my lab I use the **10.x.x.x** private range with a **/24 subnet** on each segment. A /24 gives you 256 addresses minus the network address and broadcast address — **254 usable hosts** per segment.

More importantly, the segments are isolated on purpose:

| Subnet | Purpose |
|---|---|
| Management | Command Server, Portainer, monitoring |
| Production | Vaultwarden, ntfy, Gitea, live services |
| Sandbox | Experimental VMs, anything that might break |

If something in the sandbox goes wrong, it cannot easily reach the management plane. This is a beginner version of zero-trust networking: nothing gets access just because it is connected.

---

## Switching to SQLite Changed Everything

My containers were using PostgreSQL as their database backend. PostgreSQL is great when you have many users hitting a database simultaneously. I have one user. That user is me.

A PostgreSQL container sitting idle uses around 200MB of RAM. SQLite is a single file on disk — no background daemon, no network configuration, no extra moving parts. When I moved **Vaultwarden** and **Gitea** over to SQLite backends, idle memory usage dropped by roughly **40%** and both containers started noticeably faster.

> **⚠ Know the Limits**
>
> SQLite is the right call for single-user, low-concurrency workloads. The moment you have multiple services writing simultaneously or need replication, PostgreSQL is the correct answer. Match the tool to the actual workload.

The lesson: "enterprise grade" does not automatically mean "right for your situation."

---

## Fixing My Notes Setup: The WebDAV Pivot

For a while I ran Joplin Server — a self-hosted sync service for the Joplin note-taking app. It kept throwing `SQLITE_CANTOPEN` errors whenever the storage mount had even a brief delay. The irony: I could not write down notes about fixing the problem because the note system was the broken thing.

The fix was switching to **WebDAV**, which syncs notes as plain files over HTTP. No database. No daemon. Backup is a single `rsync` command. Removing the complexity fixed the reliability completely.

This is the core principle of the lean lab applied to documentation: add complexity only when simpler solutions genuinely cannot do the job.

---

## Getting Alerts on My Phone

A lab that breaks silently is not really a lab — it is just a pile of computers. I set up **ntfy**, a lightweight notification service that works by sending simple HTTP requests. Any script that can run `curl` can fire an alert.

I also wrote a Bash script that bridges ntfy to Telegram. It long-polls the local ntfy server and forwards alerts to my phone as encrypted messages. No inbound ports required — all outbound traffic.

```bash
#!/bin/bash
# ntfy_telegram_bridge.sh
# Long-polls a local ntfy topic and forwards alerts to a Telegram bot.
# Requirements: curl, jq
# Usage: chmod +x ntfy_telegram_bridge.sh && ./ntfy_telegram_bridge.sh

NTFY_URL="http://localhost:80/your-topic/json"
BOT_TOKEN="your_telegram_bot_token"
CHAT_ID="your_chat_id"
TG_URL="https://api.telegram.org/bot${BOT_TOKEN}/sendMessage"

# Send a message to Telegram
send_telegram() {
  local message="$1"
  curl -s -X POST "$TG_URL" \
    -d "chat_id=${CHAT_ID}" \
    --data-urlencode "text=${message}" \
    > /dev/null
}

echo "[*] Bridge live. Polling ${NTFY_URL}..."

while true; do
  # Block until the server sends an event (90s timeout prevents stale hang)
  EVENT=$(curl -s --max-time 90 "${NTFY_URL}" | head -n 1)

  # Only forward actual messages — skip keepalive/open events
  EVENT_TYPE=$(echo "$EVENT" | jq -r '.event // "keepalive"')

  if [[ "$EVENT_TYPE" == "message" ]]; then
    TITLE=$(echo "$EVENT" | jq -r '.title // "Sweatt Labs Alert"')
    MSG=$(echo "$EVENT" | jq -r '.message // ""')
    PRIORITY=$(echo "$EVENT" | jq -r '.priority // "default"')

    PAYLOAD="🔔 [${PRIORITY^^}] ${TITLE}\n${MSG}"
    send_telegram "$PAYLOAD"
    echo "[+] Forwarded: ${TITLE}"
  fi

  # Brief pause to avoid hammering on empty/error responses
  sleep 2
done
```

> **ℹ Firing an Alert from Any Script**
>
> From anywhere on the host: `curl -d "Disk usage at 90%" ntfy://localhost/your-topic`
>
> Failed SSH login, high CPU temp, container restart failure — anything that can run curl can trigger a notification.

---

## Where I Am with Certifications

The longer-term goal is a T-shaped skill set: broad general IT knowledge with a deeper security specialization.

| Status | Certification |
|---|---|
| ✅ Done | Google Data Analytics Professional |
| ✅ Done | Google Project Management Professional |
| ✅ Done | Google IT Support Professional |
| 🔄 ~50% | Google Cybersecurity Professional |
| 🎯 Next | CCNA — Summer push with Network Chuck |

Proxmox is already configured to run **GNS3** and **Cisco Packet Tracer** for the CCNA work. The network segmentation built this issue is laying the mental groundwork for Cisco-level routing concepts.

---

## What I Would Tell Myself Six Months Ago

**Write everything down.** Even if it feels obvious now. You will not remember it in three weeks, and your future self will be annoyed.

**One service rule.** Keep services simple and separate. When one breaks it should not take three others with it.

**Print your infrastructure.** 3D printing bridges the gap between hobbyist chaos and disciplined physical setup for almost nothing.

**Live in the CLI.** Almost everything worth doing happens there. Learn it early and the rest gets easier.

The stack is stable now. On to the next thing.

*Stay curious. Stay secure. 🚀*

---

## Command Reference

### Docker

#### Container Lifecycle

| Command | What It Does |
|---|---|
| `docker ps -a` | List all containers — running and stopped |
| `docker stop $(docker ps -aq)` | Stop every running container at once |
| `docker start <name>` | Start a stopped container by name |
| `docker restart <name>` | Stop and start — reloads config |
| `docker rm -f <name>` | Force-delete even a running container |

#### Running Containers

| Command | What It Does |
|---|---|
| `docker run -d` | Run detached (background) |
| `docker run -p 8080:80` | Map host port 8080 → container port 80 |
| `docker run -v vol:/path` | Mount a named volume into the container |
| `docker run -e VAR=value` | Pass an environment variable |
| `docker run --restart=unless-stopped` | Auto-restart unless manually stopped |

#### Debugging

| Command | What It Does |
|---|---|
| `docker logs -f <name>` | Tail live logs from a container |
| `docker inspect <name>` | Full metadata — ports, mounts, network config |
| `docker exec -it <name> bash` | Drop into a running container's shell |
| `docker stats` | Live CPU and RAM usage per container |
| `docker system prune -a` | Nuke all stopped containers, unused images and networks |

#### Volumes & Networks

| Command | What It Does |
|---|---|
| `docker volume ls` | List all named volumes |
| `docker volume create <name>` | Create a named volume |
| `docker network ls` | List all Docker networks |
| `docker network inspect bridge` | Inspect default bridge + attached containers |

---

### Bash

#### Navigation & Files

| Command | What It Does |
|---|---|
| `ls -lah` | List with permissions, human-readable sizes, hidden files |
| `cd -` | Jump back to previous directory |
| `rsync -av src/ dest/` | Sync directories — better than cp for backups |
| `find / -name "*.conf" 2>/dev/null` | Find files by name, suppress permission errors |
| `chmod +x script.sh` | Make a script executable |

#### Text Processing

| Command | What It Does |
|---|---|
| `grep -r "pattern" /path` | Recursive string search across files |
| `grep -E ':80\|:443'` | Extended regex — match multiple patterns |
| `cat file \| jq '.'` | Pretty-print JSON output |
| `awk '{print $2}'` | Extract the second column from output |
| `tail -f /var/log/syslog` | Follow a log file live |

#### Scripting

| Syntax | What It Does |
|---|---|
| `$(command)` | Command substitution — use output as a value |
| `if [[ "$VAR" == "val" ]]; then` | Conditional — use `[[ ]]` for strings, not `[ ]` |
| `while true; do ... sleep 2; done` | Infinite loop with pause — used in the ntfy bridge |
| `curl -s --max-time 90 "$URL"` | Silent curl with timeout — essential for long-polling |
| `2>/dev/null` | Discard stderr — keeps script output clean |

#### Cron

| Syntax | What It Does |
|---|---|
| `crontab -e` | Edit current user's cron jobs |
| `0 2 * * * /path/script.sh` | Run at 2:00 AM every day |
| `*/5 * * * * /path/script.sh` | Run every 5 minutes |

---

### Python

#### HTTP Requests

| Snippet | What It Does |
|---|---|
| `import requests` | Load the requests library |
| `r = requests.get(url, timeout=10)` | GET with a 10s timeout |
| `r = requests.post(url, data={...})` | POST with form data |
| `r.json()` | Parse JSON response body to dict |
| `r.raise_for_status()` | Raise exception on 4xx or 5xx |

#### File & System

| Snippet | What It Does |
|---|---|
| `subprocess.run(["cmd","arg"], check=True)` | Run a shell command, raise error if it fails |
| `os.environ.get("VAR")` | Read environment variables safely |
| `json.loads(text)` | Parse a JSON string to dict |
| `pathlib.Path("f").read_text()` | Read a file in one line |

#### Patterns for Lab Scripts

| Pattern | What It Does |
|---|---|
| `if __name__ == "__main__":` | Entry point guard — only runs when called directly |
| `try: ... except Exception as e:` | Basic error handling — don't let scripts die silently |
| `import argparse` | Parse CLI arguments cleanly |
| `import logging; logging.basicConfig(...)` | Proper log output instead of print statements |

---

### Networking

#### Port Investigation

| Command | What It Does |
|---|---|
| `sudo ss -tulpn` | All listening ports + process holding each one |
| `sudo lsof -i :80` | What process is using port 80 |
| `sudo lsof -i -P -n \| grep LISTEN` | All listening sockets, no DNS resolution |

#### Connectivity Testing

| Command | What It Does |
|---|---|
| `curl -I http://localhost:80` | HTTP headers only — fastest way to confirm a service is up |
| `curl -v http://host:port` | Verbose — shows full connection negotiation |
| `ping -c 4 10.0.0.1` | Basic ICMP reachability check |
| `traceroute 10.0.0.1` | Show each hop to the destination |
| `nmap -sV 10.0.0.0/24` | Scan subnet for open ports and service versions |

#### iptables — Docker Context

| Command | What It Does |
|---|---|
| `sudo iptables -L -n -v` | List all rules with packet counts |
| `sudo iptables -t nat -L -n` | NAT table — where Docker's port forwarding rules live |

#### Interface Info

| Command | What It Does |
|---|---|
| `ip addr show` | All interfaces and their IP addresses |
| `ip route show` | Display the routing table |
| `ip -br addr` | Brief, readable interface summary |

---

### Systemctl

#### Service Control

| Command | What It Does |
|---|---|
| `systemctl status <service>` | Running state, recent logs, PID |
| `systemctl start <service>` | Start immediately |
| `systemctl stop <service>` | Stop immediately |
| `systemctl restart <service>` | Stop then start — reloads config |
| `systemctl enable <service>` | Auto-start on boot |
| `systemctl disable --now <service>` | Disable AND stop in one command |
| `systemctl list-units --type=service` | List all active services |
| `systemctl daemon-reload` | Reload systemd after editing unit files |

#### Logs via journald

| Command | What It Does |
|---|---|
| `journalctl -u <service> -f` | Follow live logs for a specific service |
| `journalctl -u <service> --since "1h ago"` | Logs from the past hour |
| `journalctl -p err -b` | Only errors since last boot |
| `journalctl --vacuum-time=7d` | Delete logs older than 7 days |

---

### Proxmox

#### VM Management

| Command | What It Does |
|---|---|
| `qm list` | List all VMs and their status |
| `qm start <vmid>` | Start a VM by ID |
| `qm shutdown <vmid>` | Graceful shutdown via guest agent |
| `qm stop <vmid>` | Force stop — hard power off |
| `qm config <vmid>` | Full VM config — cores, RAM, disks, network |
| `qm set <vmid> --memory 4096` | Change RAM allocation in MB |
| `qm terminal <vmid>` | Attach to VM serial console |

#### LXC Containers

| Command | What It Does |
|---|---|
| `pct list` | List all LXC containers |
| `pct start <ctid>` | Start an LXC container |
| `pct enter <ctid>` | Open a shell inside the container |
| `pct config <ctid>` | Show container config |
| `pct set <ctid> --memory 512` | Update container memory limit |

#### Storage & Backup

| Command | What It Does |
|---|---|
| `pvesm status` | All storage backends and usage |
| `vzdump <vmid> --storage <pool>` | Backup a VM or container |
| `qmrestore <backup> <vmid>` | Restore a VM from a vzdump backup |

#### Node Info

| Command | What It Does |
|---|---|
| `pveversion` | Show Proxmox VE version |
| `pve-firewall status` | Check Proxmox firewall state |

---

### SQLite

#### Connecting

| Command | What It Does |
|---|---|
| `sqlite3 database.db` | Open or create a database file |
| `.tables` | List all tables |
| `.schema <table>` | Show CREATE statement for a table |
| `.mode column` | Format output as aligned columns |
| `.headers on` | Show column names in query output |
| `.quit` | Exit the SQLite shell |

#### Common Queries

| Query | What It Does |
|---|---|
| `SELECT * FROM table LIMIT 10;` | Preview first 10 rows |
| `SELECT COUNT(*) FROM table;` | Count all rows |
| `SELECT * FROM table WHERE col='val';` | Filter rows by value |
| `UPDATE table SET col='val' WHERE id=1;` | Update a specific row |
| `PRAGMA table_info(table);` | Show columns, types, and constraints |

#### Backup & Maintenance

| Command | What It Does |
|---|---|
| `sqlite3 db.db ".backup backup.db"` | Hot backup while the service is running |
| `sqlite3 db.db "PRAGMA integrity_check;"` | Check for corruption |
| `sqlite3 db.db "VACUUM;"` | Reclaim disk space, defragment the file |
| `sqlite3 db.db ".dump" > dump.sql` | Export entire database as SQL text |
| `sqlite3 new.db < dump.sql` | Restore from a SQL dump |
