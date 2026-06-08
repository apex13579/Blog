| Command | What It Does |
|---|---|
| `sudo ss -tulpn` | All listening ports + process holding each one |
| `sudo lsof -i :80` | What process is using port 80 |
| `sudo lsof -i -P -n \| grep LISTEN` | All listening sockets, no DNS resolution |
| `curl -I http://localhost:80` | HTTP headers only — fastest way to confirm a service is up |
| `curl -v http://host:port` | Verbose — shows full connection negotiation |
| `ping -c 4 10.0.0.1` | Basic ICMP reachability check |
| `traceroute 10.0.0.1` | Show each hop to the destination |
| `nmap -sV 10.0.0.0/24` | Scan subnet for open ports and service versions |
| `sudo iptables -L -n -v` | List all rules with packet counts |
| `sudo iptables -t nat -L -n` | NAT table — where Docker's port forwarding rules live |
| `ip addr show` | All interfaces and their IP addresses |
| `ip route show` | Display the routing table |
| `ip -br addr` | Brief, readable interface summary |

```text
# cisco_ios_baseline.txt
enable                  # Enter privileged EXEC mode
configure terminal      # Drop into global configuration mode
hostname Switch01       # Apply device naming convention
write erase             # Wipe startup config on used gear
```

```text
# interface_management.txt
show ip interface brief  # Snapshot of all interface states
show running-config      # Verify active config against your baseline
interface f0/1           # Enter config mode for FastEthernet 0/1
shutdown                 # Disable the port
no shutdown              # Re-enable and bring the link up
```

```text
# subnet_math_reference.txt
Subnet mask         = 255.255.255.0
Total address space = 256 addresses
Network address     = -1
Broadcast address   = -1
Usable hosts        = 254
```
