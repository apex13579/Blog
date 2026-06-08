### 🌐 Linux Networking & Diagnostics

| Command | What It Does | Why it Matters |
| :--- | :--- | :--- |
| `sudo ss -tulpn` | Lists all listening ports and their associated process IDs. | Crucial for identifying exactly which application or container is occupying a specific network port. |
| `sudo lsof -i :80` | Shows the exact process currently utilizing port 80. | The fastest way to troubleshoot "Port already in use" errors when launching web servers. |
| `sudo lsof -i -P -n \| grep LISTEN` | Lists all open sockets using numeric ports/IPs instead of resolving names. | Speeds up output significantly by bypassing slow DNS reverse-lookups. |
| `curl -I http://localhost:80` | Fetches HTTP headers only from a local web service. | The most efficient way to confirm a web server is up and responding without downloading full pages. |
| `curl -v http://host:port` | Runs a verbose connection test showing the full network handshake. | Ideal for debugging firewall blocks, SSL/TLS handshake failures, and proxy redirections. |
| `ping -c 4 10.0.0.1` | Sends exactly 4 ICMP echo requests to a target IP. | A quick baseline test to verify layer 3 network connectivity to a specific host. |
| `traceroute -n 10.0.0.1` | Traces the network path to a target without resolving hostnames. | PINPOINTS exactly which router or gateway hop is dropping your packets across a network. |
| `nmap -sV 10.0.0.0/24` | Scans a local subnet to discover live hosts, open ports, and software versions. | Essential tool for network inventory auditing and identifying unpatched software vulnerabilities. |
| `sudo iptables -L -n -v` | Lists active firewall rules alongside real-time packet/byte counters. | Allows you to see exactly which security rules are actively matching and dropping or passing traffic. |
| `sudo iptables -t nat -L -n` | Displays Network Address Translation rules. | Where you look to verify that Docker or podman port-forwarding mappings are running correctly. |
| `ip -br addr` | Displays a brief, columnized summary of network interfaces and IPs. | Vastly superior and easier to read than the classic, incredibly dense `ip addr show` output. |
| `ip route show` | Displays the system's local routing table. | Shows you exactly which interface your default gateway is bound to for outbound traffic. |

---

### 🎛️ Cisco IOS Management Reference

*Use these commands when accessing your switch or network hardware via a serial console connection or secure shell (SSH).*

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
| **Total IP Address Space** | `256` | $2^{(32 - 24)} = 2^8 = 256$ |
| **Network Address** | `.0` (e.g., `192.168.1.0`) | First address in the range; identifies the network itself. |
| **Broadcast Address** | `.255` (e.g., `192.168.1.255`) | Final address in the range; used to send data to all hosts on the subnet. |
| **Usable Host Space** | `254` | $\text{Total Addresses} - 2$ (Subtracts Network and Broadcast addresses). |
