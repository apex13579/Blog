# 🐧 The Ultimate Systems & Infrastructure Cheatsheet

A beginner-friendly reference guide for Linux administration, network engineering, and Docker container operations.

---

## 💻 Linux & Bash Essentials

### 📋 Core Commands

| Command | Purpose | Why it Matters |
| :--- | :--- | :--- |
| `sudo` | Super User Do | Grants admin/root privileges required to make core system changes. |
| `apt update` | Refresh Repositories | Syncs your local package list with remote servers (for Debian/Ubuntu systems). |
| `docker run` | Start Container | Deploys services. Often used with `-d` (run in background) and `-p` (map network ports). |
| `timedatectl` | Time Management | Ensures your server's logs and automated jobs execute in the correct timezone. |
| `nano filename.txt` | Text Editor | The easiest, most beginner-friendly way to edit configuration files directly in the terminal. |
| `ls -lah` | Detailed List | Shows directory contents with human-readable file sizes (MB/GB) and reveals hidden files. |
| `cd -` | Quick Jump | Instantly returns you to the last directory you were in, saving you from typing long paths. |
| `rsync -av src/ dest/` | Advanced Sync | Better than `cp` for backups. It resumes broken transfers and only copies files that have changed. |
| `find / -name "*.conf" 2>/dev/null` | Deep Search | Finds files by name across the whole drive. The `2>/dev/null` hides the annoying "Permission denied" errors. |
| `chmod +x script.sh` | Grant Execution | Changes a plain text file into an executable program that the system can run. |
| `grep -r "pattern" /path` | Inside-File Search | Recursively looks inside every file within a folder to find a specific string of text. |
| `grep -E ':80\|:443'` | Multi-Pattern Search | Uses extended regular expressions to search for multiple different strings at the same time. |
| `cat file \| jq '.'` | Format JSON | Takes messy, unreadable JSON data and formats it nicely so humans can read it. |
| `awk '{print $2}'` | Column Extractor | Pulls out only the specific column of text you need from messy terminal output. |
| `tail -f /var/log/syslog` | Live Log Viewer | Watches a file update in real-time. Essential for troubleshooting why a service won't start. |
| `$(command)` | Command Substitution | Runs a command and saves its output so you can use it inside another command or script. |
| `curl -s --max-time 90 "$URL"` | Silent Web Request | Pulls data from a URL without showing a progress bar, and times out if the server hangs. |

---

### ⚙️ Scripting & Automation Syntax

Unlike standalone terminal commands, these statements and configuration formats are designed to be used inside your custom automation scripts and system configuration files.

| Syntax / Scripting Concept | Purpose | Why it Matters |
| :--- | :--- | :--- |
| `crontab -e` | Task Scheduler | Opens the file where you configure scripts to run automatically on a set schedule. |
| `0 2 * * * /path/script.sh` | Cron (Daily) | Cron syntax telling the system: "Run this script exactly at 2:00 AM every day." |
| `*/5 * * * * /path/script.sh` | Cron (Frequent) | Cron syntax telling the system: "Run this script every 5 minutes." |
| `if [[ "$VAR" == "val" ]]; then` | Conditional Logic | Checks if a string condition is true before executing the next block of code. |
| `while true; do ... sleep 2; done` | Infinite Loop | Runs a task endlessly with a built-in pause. Perfect for creating custom notification bridges. |

---

## 🌐 Networking & Diagnostics

### 📋 Linux Networking & Triage

| Command | What It Does | Why it Matters |
| :--- | :--- | :--- |
| `sudo ss -tulpn` | Lists all listening ports and their associated process IDs. | Crucial for identifying exactly which application or container is occupying a specific network port. |
| `sudo lsof -i :80` | Shows the exact process currently utilizing port 80. | The fastest way to troubleshoot "Port already in use" errors when launching web servers. |
| `sudo lsof -i -P -n \| grep LISTEN` | Lists all open sockets using numeric ports/IPs instead of resolving names. | Speeds up output significantly by bypassing slow DNS reverse-lookups. |
| `curl -I http://localhost:80` | Fetches HTTP headers only from a local web service. | The most efficient way to confirm a web server is up and responding without downloading full pages. |
| `curl -v http://host:port` | Runs a verbose connection test showing the full network handshake. | Ideal for debugging firewall blocks, SSL/TLS handshake failures, and proxy redirections. |
| `ping -c 4 10.0.0.1` | Sends exactly 4 ICMP echo requests to a target IP. | A quick baseline test to verify layer 3 network connectivity to a specific host. |
| `traceroute -n 10.0.0.1` | Traces the network path to a target without resolving hostnames. | Pinpoints exactly which router or gateway hop is dropping your packets across a network. |
| `nmap -sV 10.0.0.0/24` | Scans a local subnet to discover live hosts, open ports, and software versions. | Essential tool for network inventory auditing and identifying unpatched software vulnerabilities. |
| `sudo iptables -L -n -v` | Lists active firewall rules alongside real-time packet/byte counters. | Allows you to see exactly which security rules are actively matching and dropping or passing traffic. |
| `sudo iptables -t nat -L -n` | Displays Network Address Translation rules. | Where you look to verify that Docker or podman port-forwarding mappings are running correctly. |
| `ip -br addr` | Displays a brief, columnized summary of network interfaces and IPs. | Vastly superior and easier to read than the classic, incredibly dense `ip addr show` output. |
| `ip route show` | Displays the system's local routing table. | Shows you exactly which interface your default gateway is bound to for outbound traffic. |

---

### 🎛️ Cisco IOS Management Reference

*Use these commands when accessing managed network hardware via a serial console connection or secure shell (SSH).*

| Command | Operational Mode | What It Does |
| :--- | :--- | :--- |
| `enable` | User EXEC | Elevates privileges to Privileged EXEC mode (enables management viewing). |
| `configure terminal` | Privileged EXEC | Drops down into Global Configuration mode to make persistent system changes. |
| `hostname Switch01` | Global Config | Assigns a unique name to the device for network identification and syslog logging. |
| `write erase` | Privileged EXEC | Completely wipes the startup configuration file, cleaning out old configurations on used hardware. |
| `show ip interface brief` | Privileged EXEC | Displays a quick snapshot of all physical ports, assigned IPs, and operational statuses. |
| `show running-config` | Privileged EXEC | Dumps the active configuration running in volatile memory to the screen. |
| `interface f0/1` | Global Config | Selects FastEthernet Port 1 to allow specific port-level changes. |
| `shutdown` | Interface Config | Administratively disables the selected physical interface port. |
| `no shutdown` | Interface Config | Powers the selected port back up and forces the physical link status active. |

---

### 🧮 Subnet Mathematics Cheat-Sheet (`/24` Example)

| Metric | Value | Reference / Formula |
| :--- | :--- | :--- |
| **CIDR Notation** | `/24` | Count of the masked network bits. |
| **Subnet Mask** | `255.255.255.0` | Binary representation of the network portion. |
| **Total IP Address Space** | `256` | 2^(32 - 24) = 2^8 = 256 |
| **Network Address** | `.0` (e.g., `192.168.1.0`) | First address in the range; identifies the network itself. |
| **Broadcast Address** | `.255` (e.g., `192.168.1.255`) | Final address in the range; used to send data to all hosts on the subnet. |
| **Usable Host Space** | `254` | Total Addresses - 2 (Subtracts Network and Broadcast addresses). |

---

## 🐳 Docker Container Management

### 🔄 Container Lifecycle

| Command | What It Does | Why it Matters |
| :--- | :--- | :--- |
| `docker ps` | Lists currently running containers. | Shows you what services are active right now and how long they have been up. |
| `docker ps -a` | Lists all containers (both running and stopped). | Essential for finding broken containers that exited or failed to start. |
| `docker stop $(docker ps -aq)` | Gracefully stops every single running container simultaneously. | The fastest way to clear the deck before performing host updates or reboots. |
| `docker start <name>` | Boots up a stopped container while preserving its previous configuration. | Brings a service back online without needing to re-type a long execution configuration. |
| `docker restart <name>` | Stops and immediately restarts a container. | Forces configuration file reloads and clears stuck application processes. |
| `docker rm -f <name>` | Force-deletes a container, even if it is currently running. | Instantly destroys a container so you can rebuild it from scratch without stopping it first. |

---

### 🚀 Running Containers

| Command | What It Does | Why it Matters |
| :--- | :--- | :--- |
| `docker run -d <image>` | Launches a container detached in the background. | Frees up your terminal window so the service runs silently as a background daemon. |
| `docker run -p 8080:80 <image>` | Maps ports using **HOST : CONTAINER** format. | Tells Docker: "When traffic hits my server's port 8080, route it to the container's port 80." |
| `docker run -v vol:/path <image>` | Mounts a persistent volume using **VOLUME : CONTAINER_PATH**. | Prevents data loss. Ensures databases or app data survive when the container is deleted or updated. |
| `docker run -e VAR=value <image>` | Injects an environment variable into the container. | Used to securely pass runtime configurations like timezones, database passwords, or API keys. |
| `docker run --restart=unless-stopped` | Sets an automatic restart policy for the container. | Ensures your services spin back up automatically if the host server reboots or crashes. |

---

### 🛠️ Troubleshooting & Diagnostics

| Command | What It Does | Why it Matters |
| :--- | :--- | :--- |
| `docker logs -f <name>` | Streams a container's internal log output in real-time. | The ultimate debugging command for watching application errors or connection attempts live. |
| `docker inspect <name>` | Dumps the full, low-level JSON metadata of a container. | Used to find internal IP addresses, exact volume paths, and environment variables. |
| `docker exec -it <name> sh` | Drops you directly into an interactive shell inside the container. | Allows you to navigate the container's internal filesystem. *Note: Use sh instead of bash for minimal/Alpine images.* |
| `docker stats` | Displays a live, streaming resource usage dashboard for all containers. | Quickly identifies memory leaks or rogue containers that are hogging CPU and RAM. |
| `docker system prune -a` | **Warning:** Nukes all stopped containers, unused networks, and cached images. | A nuclear option to instantly reclaim gigabytes of disk space when your drive fills up. |

---

### 💾 Volumes & Networking

| Command | What It Does | Why it Matters |
| :--- | :--- | :--- |
| `docker volume ls` | Lists every named storage volume managed by Docker. | Helps track down abandoned volumes that are taking up storage space after containers are deleted. |
| `docker volume create <name>` | Manually provisions an isolated storage directory. | Allows you to pre-stage persistent storage spaces before attaching them to new containers. |
| `docker network ls` | Lists all isolated virtual networks built on the host. | Lets you see what communication lanes exist (e.g., bridge, host, macvlan) for your containers. |
| `docker network inspect bridge` | Inspects the default bridge network to show its attached containers. | Shows you exactly which containers can talk to one another and lists their internal subnet IPs. |
