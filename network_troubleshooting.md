# Network Troubleshooting SOP — OSI Model Framework

## Purpose

This document establishes a standardized process for diagnosing and resolving network
issues using the OSI model as the troubleshooting framework. Following this process
consistently produces faster resolutions, reduces guesswork, and creates a repeatable
methodology that scales from a single homelab node to a production enterprise environment.

The OSI model gives every network problem a home. If you know which layer the problem
lives on, you know what to check, what commands to run, and what to rule out. This SOP
walks through every layer from the bottom up — the direction data travels when it first
enters the network.

---

## The golden rule before you start

**Check the boring stuff first.**

The vast majority of network issues are Layer 1 problems — a cable that came loose, a
port that was accidentally shut down, a device that was never plugged in. Before running
a single command, physically inspect the environment. The experienced network engineer
and the beginner both check the physical layer first. The difference is the experienced
engineer does it without embarrassment.

> **Rule:** Start in the perceived middle of the problem and test outward. If you can
> ping the gateway but not the internet, the problem is above the gateway. If you cannot
> ping the gateway, the problem is between you and it. Knowing which direction to move
> cuts troubleshooting time in half.

---

## Pre-troubleshooting checklist

Before touching any configuration, complete this checklist in order:

- [ ] Identify exactly what is not working and what is working
- [ ] Identify when the problem started and what changed before it started
- [ ] Identify how many devices are affected — one, some, or all
- [ ] Document your starting state before making any changes
- [ ] Have rollback steps ready before changing any configuration

---

## Layer 1 — Physical

**What this layer does:** Transmits raw bits over a physical medium — copper cable,
fiber, or wireless radio frequency.

**Symptoms of a Layer 1 problem:**
- Interface shows as down/down in `show ip interface brief`
- Link light is off on the switch or NIC
- No connectivity at all — cannot even ping the default gateway
- Intermittent connectivity that correlates with physical movement of cables

**Troubleshooting steps:**

1. Check all cable connections at both ends — reseat if any doubt exists
2. Verify link lights are active on both the switch port and the device NIC
3. Swap the cable with a known-good cable
4. Try a different switch port
5. Check for physical damage — bent pins, damaged RJ45 connectors, kinked fiber
6. For wireless — verify the device is within range and the WAP is powered on
7. Verify the interface is not administratively shut down

**Commands:**

```text
show ip interface brief          # Look for down/down status
show interfaces [int]            # Check for input/output errors, CRC errors
show interfaces status           # Speed, duplex, and connection state
```

**Resolution criteria:** Link lights are active on both ends, interface shows up/up,
and physical errors are zero or negligible.

---

## Layer 2 — Data Link

**What this layer does:** Handles MAC address-based frame forwarding between devices
on the same network segment. Switches live here.

**Symptoms of a Layer 2 problem:**
- Interface shows up/up but no traffic passes
- Device can be pinged from directly connected devices but not across the network
- Duplicate IP warnings or unexpected ARP behavior
- VLAN misconfiguration — device appears on wrong network segment

**Troubleshooting steps:**

1. Verify the correct VLAN is assigned to the switch port
2. Check the CAM table to confirm the device MAC address is learned on the expected port
3. Verify trunk ports are carrying the correct VLANs
4. Check for duplex mismatches — one side full duplex, other side half duplex
5. Look for excessive CRC errors or input errors indicating a bad cable or NIC
6. Verify Spanning Tree is not blocking the port unexpectedly
7. If the port leads to another switch, trace the MAC address hop by hop

**Commands:**

```text
show mac address-table                       # Full CAM table
show mac address-table interface [port]      # CAM entries for one port
show mac address-table address [mac]         # Find which port a MAC is on
show interfaces [int]                        # Check duplex, speed, errors
show interfaces trunk                        # Verify trunk port VLAN allowances
show spanning-tree                           # Check for blocked ports
show vlan brief                              # Verify VLAN assignments
clear mac address-table dynamic              # Clear learned MAC entries
```

**Resolution criteria:** Device MAC appears in CAM table on the correct port, correct
VLAN is assigned, duplex and speed match on both ends, and no excessive errors.

---

## Layer 3 — Network

**What this layer does:** Handles IP address-based routing between networks. Routers
live here. This is where broadcast domains end and routing decisions are made.

**Symptoms of a Layer 3 problem:**
- Can ping devices on the same subnet but not devices on other subnets
- Routing table is missing expected routes
- Wrong default gateway configured on the device
- NAT not translating correctly — internal devices cannot reach the internet

**Troubleshooting steps:**

1. Verify the device has the correct IP address, subnet mask, and default gateway
2. Ping the default gateway — if this fails, the problem is Layer 1 or 2
3. Ping a known IP address on a remote network — if this fails, check routing
4. Check the routing table for missing or incorrect routes
5. Verify NAT is configured correctly on the gateway router
6. Check for ACLs that may be blocking traffic
7. Verify the subnet mask is correct — a wrong subnet mask is a common silent failure

**Commands:**

```text
show ip route                              # Full routing table
show ip route [network]                    # Specific route lookup
show ip interface brief                    # IP addresses on all interfaces
show arp                                   # IP-to-MAC mappings
show ip nat translation                    # Active NAT translations
show ip access-lists                       # Check for blocking ACLs
ping [ip address]                          # Basic connectivity test
ping [ip] source [interface]               # Ping from a specific interface
traceroute [ip]                            # Trace the path to destination
```

**On Windows/Linux endpoints:**

```bash
ipconfig /all          # Windows — full IP configuration including gateway and DNS
ifconfig               # Linux/Mac — interface configuration
ip addr                # Linux — modern interface configuration
ip route               # Linux — routing table
arp -a                 # Windows/Linux — ARP cache
ping [ip]              # Basic connectivity
tracert [ip]           # Windows — trace route
traceroute [ip]        # Linux — trace route
```

**Resolution criteria:** Device has correct IP, subnet mask, and gateway. Can ping
gateway. Routing table has correct routes. NAT translations appear for outbound traffic.

---

## Layer 4 — Transport

**What this layer does:** Manages end-to-end communication between applications.
TCP provides reliable ordered delivery. UDP provides fast connectionless delivery.
Port numbers live here.

**Symptoms of a Layer 4 problem:**
- Can ping a device but cannot connect to a specific service on it
- Connection timeouts on specific ports
- Application connects but behaves erratically — indicative of packet loss
- TCP handshake not completing

**Troubleshooting steps:**

1. Verify the specific port is open on the destination device
2. Check firewall rules on both the source and destination — a firewall blocking
   a port is the most common Layer 4 issue
3. Verify the service is actually running and listening on the expected port
4. Check for ACLs on network devices filtering specific port traffic
5. Test the TCP handshake explicitly using telnet or netcat
6. Look for packet loss using extended ping — loss above 1% indicates a problem

**Commands:**

```bash
# Test if a specific port is open
telnet [ip] [port]                          # Windows/Linux
nc -zv [ip] [port]                          # Linux — netcat port test

# Check what is listening on which ports
netstat -an                                 # All connections and listening ports
ss -tulnp                                   # Linux — socket statistics
```

```text
# Cisco IOS
show ip access-lists                        # Check for port-based ACL filtering
show tcp brief                              # Active TCP connections
```

**Resolution criteria:** Target port is open and accepting connections. TCP handshake
completes successfully. No firewall rules blocking the specific port.

---

## Layer 5 — Session

**What this layer does:** Establishes, maintains, and terminates communication sessions
between applications. Authentication and session management live here.

**Symptoms of a Layer 5 problem:**
- Authentication failures despite correct credentials
- Sessions dropping unexpectedly and requiring re-authentication
- VPN tunnels dropping and not re-establishing
- Remote desktop or SSH sessions disconnecting mid-use

**Troubleshooting steps:**

1. Verify credentials are correct and the account is not locked
2. Check session timeout settings on the server or application
3. Verify VPN tunnel configuration matches on both ends — mismatched parameters
   cause sessions to drop after the initial negotiation period
4. Check for NAT timeout issues dropping long-lived sessions
5. Review authentication logs on the server for rejection reasons

**Commands:**

```bash
# SSH session debugging
ssh -v [user]@[ip]                          # Verbose SSH — shows session negotiation

# Check authentication logs
tail -f /var/log/auth.log                   # Linux — authentication log
journalctl -u sshd                          # Linux — SSH service log
```

**Resolution criteria:** Authentication succeeds. Sessions maintain stability for the
expected duration. VPN tunnels establish and hold without intervention.

---

## Layer 6 — Presentation

**What this layer does:** Handles data formatting, encryption, and compression.
TLS/SSL certificates live here.

**Symptoms of a Layer 6 problem:**
- SSL certificate errors in the browser
- Expired or self-signed certificate warnings
- Encryption negotiation failures
- Data appearing corrupted or garbled after transmission

**Troubleshooting steps:**

1. Check SSL certificate validity — expiration date, domain name match, issuing CA
2. Verify TLS version compatibility between client and server — older clients may
   not support TLS 1.3, newer servers may have dropped TLS 1.0 support
3. Check for certificate chain issues — intermediate certificates missing
4. Verify time synchronization on both devices — certificate validation depends on
   accurate time and a clock that is significantly off will cause certificate failures

**Commands:**

```bash
# Check certificate details
openssl s_client -connect [domain]:443      # Full TLS handshake details
curl -vI https://[domain]                   # Certificate info in curl output

# Check time synchronization
timedatectl                                 # Linux — time and NTP status
w32tm /query /status                        # Windows — time sync status
```

**Resolution criteria:** Certificate is valid, not expired, matches the domain name,
issued by a trusted CA, and the full certificate chain is present.

---

## Layer 7 — Application

**What this layer does:** The layer the user interacts with directly. HTTP, DNS, DHCP,
SMTP, FTP, and all application-level protocols live here.

**Symptoms of a Layer 7 problem:**
- Can connect to a server but the application returns errors
- DNS resolution failing — can ping by IP but not by hostname
- DHCP not assigning addresses
- Web application returning 404, 500, or other HTTP errors
- Email not sending or receiving

**Troubleshooting steps:**

1. Test DNS resolution explicitly — can you resolve the hostname to an IP?
2. If DNS fails, check the DNS server configuration and verify the DNS server
   is reachable
3. If DHCP is not assigning addresses, check the DHCP scope for exhaustion
4. Check application-level logs for error messages — Layer 7 problems almost
   always leave specific error messages that point directly to the cause
5. Verify the application service is running
6. Check for application firewall rules separate from network firewall rules

**Commands:**

```bash
# DNS troubleshooting
nslookup [hostname]                         # Basic DNS lookup
nslookup [hostname] [dns-server]            # Query a specific DNS server
dig [hostname]                              # Linux — detailed DNS lookup
dig [hostname] @[dns-server]               # Query a specific DNS server

# DHCP troubleshooting
ipconfig /release && ipconfig /renew        # Windows — release and renew DHCP
dhclient -r && dhclient                     # Linux — release and renew DHCP

# HTTP troubleshooting
curl -I [url]                               # HTTP headers and response code
curl -v [url]                               # Verbose HTTP request

# Check if a service is running
systemctl status [service]                  # Linux — service status
```

```text
# Cisco IOS
show ip dhcp pool                           # DHCP pool configuration
show ip dhcp binding                        # Active DHCP leases
show ip dhcp conflict                       # IP address conflicts
debug ip dhcp server events                 # Real-time DHCP debugging
```

**Resolution criteria:** DNS resolves correctly. DHCP assigns addresses from the
correct scope. Application service is running and returning expected responses.

---

## Complete troubleshooting workflow

Use this workflow for any network problem. Work from Layer 1 upward and stop at the
layer where the problem is confirmed. Do not skip layers.

```text
STEP 1 — DEFINE THE PROBLEM
  What exactly is not working?
  What is still working?
  When did it start?
  What changed before it started?
  How many devices are affected?

STEP 2 — LAYER 1 (Physical)
  Are all cables seated and link lights active?
  Is the interface up/up in show ip interface brief?
  → If down/down: fix the physical connection
  → If up/up: proceed to Layer 2

STEP 3 — LAYER 2 (Data Link)
  Is the device MAC address in the CAM table on the correct port?
  Is the correct VLAN assigned?
  Are there duplex mismatches or excessive errors?
  → If MAC missing or wrong VLAN: fix the switch configuration
  → If MAC correct: proceed to Layer 3

STEP 4 — LAYER 3 (Network)
  Does the device have the correct IP, subnet mask, and gateway?
  Can the device ping its default gateway?
  Is there a route to the destination in the routing table?
  Is NAT translating correctly?
  → If no ping to gateway: check subnet and gateway config
  → If ping to gateway works but not beyond: check routing and NAT
  → If routing is correct: proceed to Layer 4

STEP 5 — LAYER 4 (Transport)
  Is the destination port open?
  Is a firewall blocking the specific port?
  Is the service listening on the expected port?
  → If port blocked: update firewall rules
  → If service not listening: start the service
  → If port open: proceed to Layer 5

STEP 6 — LAYER 5 (Session)
  Are credentials correct and the account active?
  Is the session timing out prematurely?
  Are VPN tunnels establishing and holding?
  → If authentication failing: check credentials and account status
  → If sessions dropping: check timeout and NAT settings
  → If sessions stable: proceed to Layer 6

STEP 7 — LAYER 6 (Presentation)
  Is the SSL certificate valid and not expired?
  Does the certificate match the domain?
  Is time synchronized correctly on both devices?
  → If certificate invalid: renew or correct the certificate
  → If time mismatch: sync NTP
  → If encryption correct: proceed to Layer 7

STEP 8 — LAYER 7 (Application)
  Does DNS resolve the hostname correctly?
  Is the application service running?
  What do the application logs say?
  → Read the error message. Layer 7 problems tell you exactly what is wrong.
     Believe them.
```

---

## Documentation requirements

Every troubleshooting session must be documented before it is closed. The record
must include:

```text
Date and time of the issue
Devices affected
Symptoms observed
Layer where the problem was found
Root cause identified
Steps taken to resolve
Resolution confirmed by
Any follow-up actions required
```

Documentation is not optional. The issue that happened once will happen again. The
engineer who documented it the first time resolves it in minutes the second time.
The engineer who did not document it starts from Layer 1 again.

---

## Quick reference — commands by layer

### Layer 1 — Physical
```text
show ip interface brief
show interfaces [int]
show interfaces status
```

### Layer 2 — Data Link
```text
show mac address-table
show mac address-table interface [port]
show mac address-table address [mac]
show interfaces [int]
show interfaces trunk
show spanning-tree
show vlan brief
clear mac address-table dynamic
show cdp neighbors
show cdp neighbors detail
```

### Layer 3 — Network
```text
show ip route
show ip interface brief
show arp
show ip nat translation
show ip access-lists
ping [ip]
ping [ip] source [int]
traceroute [ip]
```

### Layer 4 — Transport
```text
show ip access-lists
show tcp brief
telnet [ip] [port]        # test port connectivity
netstat -an               # endpoint — active connections
```

### Layer 5 — Session
```text
ssh -v [user]@[ip]
tail -f /var/log/auth.log
journalctl -u sshd
```

### Layer 6 — Presentation
```text
openssl s_client -connect [domain]:443
curl -vI https://[domain]
timedatectl
```

### Layer 7 — Application
```text
nslookup [hostname]
dig [hostname]
curl -I [url]
curl -v [url]
systemctl status [service]
show ip dhcp pool
show ip dhcp binding
show ip dhcp conflict
ipconfig /release && ipconfig /renew    # Windows
dhclient -r && dhclient                 # Linux
```

### Endpoint — Windows
```text
ipconfig /all
arp -a
tracert [ip]
netstat -an
nslookup [hostname]
ping [ip]
```

### Endpoint — Linux
```text
ip addr
ip route
arp -a
traceroute [ip]
ss -tulnp
dig [hostname]
ping [ip]
```
