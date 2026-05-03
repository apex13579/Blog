# 🚀 Home Lab Notes: Stabilizing the Stack
**Issue 3 — From Chaos to Clean Architecture**
**Date:** May 2, 2026

---

### 🎯 The Mission: Destroying the "Old Gear" Excuse
This isn't just a maintenance log; it’s a **manifesto**. The core philosophy of this lab-building exercise is to create a **fully functional, professional-grade environment** that is as lightweight as humanly possible. 

For too long, the barrier to entry in cybersecurity and network engineering has been the "hardware wall." We are taught that you need power-hungry enterprise racks and noisy server blades to do "real" work. This project exists to prove that is a myth. By prioritizing free, open-source software and efficient backends like **SQLite** and **WebDAV**, we are effectively eliminating the excuse that "my hardware is too old." 

Whether you are running on a high-end workstation or a $20 thrift-store laptop, this stack is designed to be lean, mean, and accessible to everyone. No gatekeeping, just pure engineering. We are proving that you don't need a massive power bill or the latest enterprise silicon to run a sophisticated, secure lab. This is about democratization of knowledge through technical efficiency. We are building a curriculum that anyone can follow, regardless of their budget.

---

### 🛠️ The Great Network Restoration: Diagnosing the Ghost in the Machine
The symptoms were classic homelab "gremlins": **Vaultwarden** wouldn't load, **ntfy** refused to bind to port 80, and **Portainer** went dark. 

#### The Problem: Port Squatting & Stale Docker States
Host-level processes were "squatting" on ports intended for Docker. Even after killing those processes, Docker’s internal networking layer (specifically `docker-proxy`) stayed in a stale state, believing the ports were still occupied. 

#### The Fix: The Nuclear Reset
I had to perform a full "nuclear" reset:
1.  **Stop all containers:** `docker stop $(docker ps -aq)`
2.  **Clear the host ports:** Using `sudo lsof -i :80` to find PIDs and `kill -9`.
3.  **Restart Docker:** `sudo systemctl restart docker`. This forces Docker to rebuild the `iptables` rules that handle container routing.

Suddenly, `docker-proxy` claimed port 80. A quick `curl -I` test confirmed: **HTTP 200 OK**.
### 💡 SQLite & WebDAV: The Joplin Breakthrough
In the quest for a "PostgreSQL-free" lab, I moved the stack to **SQLite**. While Gitea and Vaultwarden handled the transition beautifully, **Joplin** proved to be a stubborn beast with `SQLITE_CANTOPEN` errors.

#### The Pivot to WebDAV
I realized that for note-taking, I didn't need the complexity of a database at all. By pivoting to **WebDAV** for synchronization, I bypassed the overhead of a full database server. This setup is fast, stable, and keeps the resource footprint so low that the hardware doesn't even break a sweat.

---

## 📂 The Technical "Cheat-Code" Vault
*Note: Use these to automate the boring stuff so you can focus on the hard concepts.*

### 🐚 Bash & System Essentials
| Task | Command | Description |
| :--- | :--- | :--- |
| **Check System Load** | `uptime` | CPU load over 1, 5, and 15 mins. |
| **Memory Usage** | `free -h` | Human-readable RAM/Swap usage. |
| **Disk Usage** | `df -h` | Check if logs filled your partitions. |
| **Active Listeners** | `sudo ss -tulpn` | Essential for port conflict ID. |

### 🐳 Docker & Networking Diagnostics
*   **List All Containers:** `sudo docker ps -a` 
*   **View Live Logs:** `sudo docker logs -f <container_name>` 
*   **DNS Fix:** Edit `/etc/docker/daemon.json` and add: `{ "dns": ["8.8.8.8", "8.8.4.4"] }`.
*   **One-Line SQLite Backup:** 
    `sudo mkdir -p ~/backups/$(date +%Y%m%d) && sudo cp /opt/vaultwarden/data/db.sqlite3 ~/backups/$(date +%Y%m%d)/ 2>/dev/null`

### 🟦 Windows PowerShell Security Audit
```powershell
# Identify every local user and check Admin group membership
Get-LocalUser
Get-LocalGroupMember -Group "Administrators"

# Audit the local password policy
net accounts

# Inspect NTFS folder permissions
Get-Acl -Path "C:\Data" | Format-Table IdentityReference, FileSystemRights, AccessControlType

### 🧠 Knowledge Drop: Networking & IP Fundamentals
As I dive deeper into my studies, I’m constantly returning to the fundamentals. If you can't explain the basics, you can't secure the complex.

#### The OSI Model: A Comprehensive Deep Dive
The OSI model is the roadmap for how data moves across a network.



1.  **Physical (Layer 1):** The hardware. Cables and pulses of light/electricity.
2.  **Data Link (Layer 2):** Switching. MAC addresses live here. Data = **Frames**.
3.  **Network (Layer 3):** Routing. IP addresses live here. Data = **Packets**. 
4.  **Transport (Layer 4):** Reliability. TCP vs UDP. Data = **Segments**. Ports live here.
5.  **Session (Layer 5):** The "handshake." Manages the conversation.
6.  **Presentation (Layer 6):** The translator. Data encryption and formatting.
7.  **Application (Layer 7):** The UI. Protocols like HTTP, SSH, and SMTP.

#### IP Addressing, Classes, and Subnetting
An IP address consists of 4 octets. **Example:** `169.224.16.32` is binary `10101001.11100000.00010000.00100000`. No individual octet can ever exceed **255**.



| Class | Range | Use Case | Private Range | Subnet Mask |
| :--- | :--- | :--- | :--- | :--- |
| **Class A** | 0-127 | Huge ISPs | 10.0.0.0/8 | 255.0.0.0 |
| **Class B** | 128-191 | Medium Orgs | 172.16.0.0/12 | 255.255.0.0 |
| **Class C** | 192-223 | Home/Small Office | 192.168.0.0/16 | 255.255.255.0 |
| **Class D** | 224-239 | Multicasting | N/A | N/A |
| **Class E** | 240-254 | Experimental | N/A | N/A |
### 📡 The "Command Server" Evolution
I’ve officially transitioned my hodgepodge VM into a "Command Server" architecture—a centralized hub that manages everything else in the lab.

*   **Portainer:** My "Control Center" for all containers.
*   **Flame:** A beautiful dashboard that auto-discovers services.
*   **Tailscale:** Secure remote access using a WireGuard-based tunnel.

#### 🔗 The Telegram Notification Bridge
I built a bridge script to forward `ntfy` alerts directly to my Telegram for real-time mobile notifications:
```bash
#!/bin/bash
# ntfy-telegram-bridge.sh
while true; do
  curl -s -N "http://localhost/$NTFY_TOPIC/json" | while read line; do
    if echo "$line" | grep -q '"message"'; then
      MESSAGE=$(echo "$line" | grep -o '"message":"[^"]*"' | cut -d'"' -f4)
      curl -s -X POST "[https://api.telegram.org/bot$BOT_TOKEN/sendMessage](https://api.telegram.org/bot$BOT_TOKEN/sendMessage)" \
           -d "chat_id=$CHAT_ID" -d "text=🚨 LAB ALERT: $MESSAGE"
    fi
  done
  sleep 1
done

### 📈 Major Milestone Update: The Certification Grind
The last few months have been incredibly productive on the academic front. Here is the current progress:

*   **Google Data Analytics:** ✅ **COMPLETED!**
*   **Google Project Management:** ✅ **COMPLETED!** (Applying Agile to my lab updates).
*   **Canvas LMS:** ✅ **CERTIFIED!** Mastery for professional/volunteer education.
*   **Google Cybersecurity:** ⏳ **50% COMPLETE.** Currently mastering Python for Security.
*   **Upcoming:** Officially signed up for the **Network Chuck Summer of CCNA**! 

---

### 🏁 Final Lab State & Advice
The lab is finally in a state of "restful stability." 
*   **Virtualization:** Proxmox with auto-start enabled for the Command Server.
*   **Storage:** Expanded the primary VM to a spacious 125GB.
*   **Persistence:** All data in `/opt/*/data` (SQLite/Flat files), backed up nightly.

**Final Advice for Aspiring Lab Builders:**
1.  **Document Everything:** Write it down at 2 AM or you'll forget it by 2 PM.
2.  **Start Small:** Start with one service (like Pi-Hole) and build outward.
3.  **3D Print Your Rack:** Print your own cable management and patch panels. Stop paying for plastic; print it and buy more RAM instead!
4.  **Embrace Failure:** Troubleshooting is 90% of the job in IT. Every error is a lesson.

Got questions? Drop into the **Discord**. We’ll figure it out together.

**Stay curious. Stay secure. Happy self-hosting!** 🚀
