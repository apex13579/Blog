---
title: "Burning it down to build it right: lab rebuild, UPS wars, and the summer of CCNA"
date: 2026-07-01
category: Infrastructure & Networking
tags: [HomeLab, TrueNAS, Proxmox, NUT, AdGuard, Immich, CCNA, Network+, Python, VLANs, NAT, Subnetting]
---

## The catalyst

Four months in, the homelab had drifted. The vision had grown faster than the architecture could keep up with, services were scattered across machines without a clean rationale, and the network was still flat in places where it had no business being flat. The only honest move was a controlled demolition — blow up the old stack, document everything on the way back up, and use the rebuild as a forcing function to wire together everything being studied in the CCNA and Google Cybersecurity courses into something real. This post covers the infrastructure teardown and rebuild, a brutal UPS driver battle, the first services coming back online, and the networking and security concepts that finally clicked along the way.

---

## The teardown

Blowing up a homelab you have been running for months is uncomfortable. Every service that goes dark represents something you have to rebuild from scratch, reconfigure from memory, and document properly this time. The discomfort is the point. A lab you cannot tear down and rebuild from documentation is not a lab — it is technical debt with a power strip.

The rebuild sequence was deliberate: NAS apps first, then LXCs, then VMs, then standalone devices. Storage is the foundation everything else depends on so it goes in first. Services that talk to storage go in second. Everything else follows in dependency order.

> **Warning:** Starting a lab rebuild without a documented dependency order guarantees you will spin up services that cannot connect to the things they need and spend two hours troubleshooting a problem that was actually just sequencing.

---

## TrueNAS apps — building the storage layer first

Both Dell PowerEdge R610s were wiped and reconfigured fresh. Rather than running services as Proxmox LXCs competing for resources with the hypervisor, all NAS-resident software now runs as TrueNAS SCALE native apps. The storage layer has its own failure domain separate from the compute layer.

The first app up was Immich for photo backup — the primary motivation being getting family photos off of Apple's infrastructure and onto hardware I control. Dataset structure matters here and getting it wrong means the app will not initialize correctly.

```text
applications/
└── immich/
    ├── pgdata/     # type: generic — PostgreSQL data
    └── data/       # type: apps — required for correct permissions
```

> **Danger:** The `data` dataset must be set to type `apps` rather than `generic`. This is a TrueNAS-specific permission requirement. If you set it to generic, Immich silently fails to initialize and the logs will send you in the wrong direction.

After Immich came Uptime Kuma for service monitoring — populated with a running device list immediately on install rather than left empty to fill in later. Then Tailscale for mesh VPN access. I refuse to open ports and Tailscale handles NAT traversal automatically so remote SSH access into the network requires exactly zero firewall rule changes.

WebDAV was evaluated as a Joplin sync gateway but abandoned after the app provided no mechanism to set credentials through its interface. It goes in the column of things that will be revisited once Nextcloud is running.

> **Win:** Starting every new service installation by immediately adding its IP and port to the addressing list takes thirty seconds and has saved hours of troubleshooting during future changes. Do it at install time without exception.

---

## The NUT war — Vertiv GXT5 and a patched driver

This was the longest battle of the rebuild and the one with the most to document for anyone else who runs into it.

The Vertiv GXT5 uses USB HID for UPS monitoring but its HID data lives on interface 1 rather than the default interface 0 that standard NUT drivers target. Every standard driver connected cleanly and returned nothing useful because it was talking to the wrong interface.

The initial attempt ran NUT inside an Alpine LXC. After getting the cgroup2 device rules configured and the USB device appearing to pass through correctly, the LXC still could not reliably enumerate the device. After hours of troubleshooting the decision was made to move to a Linux Mint VM with USB passthrough configured through the Proxmox GUI instead.

> **Win:** VMs handle USB passthrough more reliably than LXCs because they present a full virtual USB controller to the guest. When a service requires direct hardware access — USB, serial, or anything else physical — use a VM. Save LXCs for network services and pure software workloads.

The driver fix came from a patched NUT 2.8.2 build on Gitea (`ftfy/nut-liebert-psa5`) with two critical configuration additions: `usb_hid_rep_index = 1` to target interface 1, and `pollonly` to work around interrupt endpoint conflicts that caused the standard driver to hang.

One additional quirk worth documenting: the GXT5 reports a 37% battery charge value when running on bypass. Shutdown thresholds must be set below 37% or every bypass event triggers a false low-battery shutdown. This is the kind of detail that gets forgotten and causes an outage six months later.

---

## AdGuard Home LXC

AdGuard Home replaced Pi-hole for DNS filtering and ad blocking. The cleaner interface, per-client statistics, and native DoH/DoT upstream support made it the obvious choice. The LXC runs Debian 12 — Debian 13 images for Proxmox have reported stability issues in the community and the risk of rebuilding a fresh LXC is not worth it this early in the stack.

The LXC requires passthrough to a physical network interface. Without it, AdGuard cannot bind correctly to the interface it needs to serve DNS. This is easy to miss during LXC creation and results in a service that appears to run but cannot actually intercept DNS queries.

Installation is a single curl command that handles binary download, systemd service creation, and initial startup automatically. Access the setup UI on port 3000 immediately after install — the service binds to port 53 after initial setup is complete.

---

## Falco and Portainer agent — infrastructure you install before you need it

A Portainer agent was installed on the NUT VM in preparation for a Portainer instance that does not exist yet. A Falco instance was installed directly on the Proxmox hypervisor rather than inside a VM or LXC.

Both of these decisions follow the same principle: infrastructure touchpoints that are installed during the initial build cost almost nothing. Infrastructure touchpoints retrofitted after the fact cost significantly more time and sometimes require service restarts that could have been avoided.

Falco at the hypervisor level watches the kernel syscall stream for all guests simultaneously. A Falco instance inside a single VM or LXC only sees that one guest's activity. The log output feeds to the SIEM over the management VLAN once that service is initialized.

---

## VLAN architecture — subnets with a purpose

The network rebuild established a clean VLAN segmentation strategy. Every VLAN maps to exactly one broadcast domain and one IP subnet. A router is required for any traffic between VLANs — this is the enforcement mechanism that makes VLAN-based security meaningful rather than decorative.

```text
VLAN 5   — Infrastructure management    /29
VLAN 10  — Management                   /27
VLAN 11  — Remote ingress               /30
VLAN 15  — Trusted workstation          /30
VLAN 20  — Bug bounty                   /30
VLAN 30  — Guest                        /28
VLAN 40  — Media                        /29
VLAN 45  — DNS server                   /30
VLAN 50  — Camera                       /28
VLAN 60  — Printer                      /29
VLAN 70  — Storage                      /29
VLAN 80  — AI                           /30
VLAN 90  — Home automation              /28
VLAN 99  — Tar pit / honeypot           /30
```

The subnets are intentionally tight by design — minimal blast radius if any VLAN is compromised. The management VLAN runs a /27 because that segment carries the most devices. Every other VLAN is sized exactly to its realistic device count with minimal headroom. This is not academic subnetting — this is security through constraint.

According to 802.1Q there can be 4096 VLANs on a single network. Trunk ports carry tagged traffic between switches using 802.1Q encapsulation. When a frame crosses a trunk port, a tag is added identifying which VLAN it belongs to. The receiving switch reads the tag and forwards accordingly.

---

## Layer 2 troubleshooting — knowing your network by its MAC addresses

One of the most valuable troubleshooting skills to build early is being able to identify every device on the network by its MAC address. A running MAC address table, a running IP address table, and a port table together form the foundation of network documentation that actually helps during an outage.

A MAC address is a 12-character hexadecimal number using digits 1-9 and letters A-F. The first six characters are the OUI (Organizationally Unique Identifier) which maps directly to the manufacturer. The second six characters uniquely identify the specific device. When hunting down an unknown device, a public OUI lookup immediately tells you whether you are looking at a Dell server, a Ubiquiti WAP, or a Canon printer that has no business being on that VLAN.

Switches operate at Layer 2 and forward frames based on MAC addresses, not IP addresses. The CAM table (Content Addressable Memory) is where the switch stores its learned MAC-to-port mappings. Following a device across a switched network means tracing the CAM table hop by hop. Multiple MAC addresses on a single port indicate a downstream switch — one port representing an entire world of devices behind it. A network diagram makes this trace dramatically faster.

ARP (Address Resolution Protocol) connects Layer 3 to Layer 2. Before any packet can be sent, the sender must know the destination MAC address. If it does not, it broadcasts a request using all-F destination MAC (`FF:FF:FF:FF:FF:FF`) asking who owns the target IP. Only after ARP resolves the MAC can the frame be properly addressed and forwarded.

**Layer 2 troubleshooting workflow:**

```text
1. Start with the IP address
2. Generate traffic with ping to populate the ARP cache
3. show arp — find the MAC address tied to the IP
4. show mac address-table — find which port that MAC is on
5. If the port leads to another switch, show cdp neighbors
6. Move to the next switch and repeat until you hit the endpoint
```

---

## Routing, EIGRP, and dynamic routing

Routers have two jobs: connecting networks and stopping broadcasts. Every router interface is its own network. Unlike switches, routers do not forward broadcast traffic — this boundary is what makes network segmentation meaningful.

EIGRP (Enhanced Interior Gateway Routing Protocol) allows routers to automatically share routing table information with neighbors in the same autonomous system, eliminating the need to manually configure every route on every router.

When two routers are configured with the same EIGRP autonomous system number and their network statements match, they form an adjacency — they recognize each other as routing partners and begin exchanging topology information. Routes learned via EIGRP appear in the routing table marked with `D`.

**Reading a routing table entry:**

```text
D 192.168.3.0/24 [90/5376] via 10.0.0.1
```

The `90` is the administrative distance — how believable the route is on a scale of 0 to 255 where lower is better, like a golf score. The `5376` is the metric — path quality used to break ties when the AD is equal. `via 10.0.0.1` is the next hop. The routing table is the brain of the router and will be the place you spend the most time troubleshooting.

---

## NAT — translating private to public

NAT (Network Address Translation) converts private inside addresses to public outside addresses so internal devices can communicate with the internet. Understanding the four NAT address types ends most NAT confusion permanently.

```text
Inside local   — your device's private IP address (you own it, it is private)
Inside global  — your device's public IP as seen from the internet (you own it, it is public)
Outside global — the internet server's public IP (they own it, it is public)
Outside local  — the internet server's address as represented inside your network (they own it, it is private)
```

PAT (Port Address Translation), also called NAT overload, maps many inside local addresses to a single inside global address using port numbers to track individual connections. This is the standard form of NAT in virtually every home and small business network.

> **Win:** If internal devices can reach each other but cannot reach the internet, and the router can ping the ISP, NAT is almost certainly the problem — not ACLs, not routing. Check NAT first.

---

## ACLs — filtering traffic with precision

ACLs filter traffic based on matching criteria applied to packets. Standard ACLs match on source IP address only and carry numbers 1-99. Extended ACLs match on source IP, destination IP, protocol, and port number and carry numbers 100-199. Standard ACLs apply close to the destination. Extended ACLs apply close to the source.

The wildcard mask is the inverse of the subnet mask — where a subnet mask uses 1 bits to identify network bits, a wildcard mask uses 0 bits for bits that must match and 1 bits for bits that can be anything.

ACLs process top-down and stop at the first match. An implicit deny-all lives at the end of every ACL. An ACL does nothing until it is applied to an interface with a direction — inbound or outbound. This is the step most commonly forgotten during initial configuration.

---

## Subnetting — the three steps that make it mechanical

Subnetting clicked when it stopped being an academic exercise and started being the thing that determines how much blast radius a compromised device has. Smaller subnets mean smaller problems.

**Step 1 — Determine hosts needed and convert to binary**

Find the minimum number of bits using `2^n - 2 ≥ hosts needed`. The bit chart from left to right: `128 | 64 | 32 | 16 | 8 | 4 | 2 | 1`. Count positions from the right until the value covers the host count.

**Step 2 — Reserve bits in the mask and find the increment**

Start from the Class C baseline (`255.255.255.0` = `/24`). Add network bits from the left of the host portion. The increment is the value of the rightmost network bit converted back to decimal.

**Step 3 — Use the increment to find network ranges**

Add the increment to find each successive subnet. The first and last address in every subnet are reserved — network address and broadcast address respectively.

> **Win (credit: Anthony Norrell):** Think of /24 as a whole pizza with 256 slices. Every time you increase the CIDR by 1, you cut the pizza in half. Every time you decrease it by 1, you double the slices. /25 = 128. /23 = 512. No chart required — just remember the formula and the doubling relationship.

**Transfer time math:**

Bytes measure storage. Bits measure network speed. One byte equals eight bits. To predict transfer time: convert file size to bits, divide by link speed in bits per second, add 20% overhead for reality.

---

## Python for security automation — Google Cybersecurity module 7

The final module of the Google Cybersecurity Certificate introduced Python as a security automation tool. After finishing the certificate — which took significantly longer than expected given the dry SQL modules in the middle — the Python content was the most directly applicable section to real lab work.

The core data types that matter most for security scripting are strings (IP addresses, hostnames, log entries), integers (port numbers, counts), booleans (access granted or denied, service up or down), and lists (device inventories, blocklists, interface collections).

Conditional statements using comparison operators (`==`, `!=`, `>`, `<`, `>=`, `<=`) are the foundation of automated decision-making — checking whether a battery charge is below a threshold, whether a service is responding, whether an IP appears on a blocklist. Iterative statements handle repetitive tasks across device lists and log files without manual intervention.

> **Warning:** The Google Cybersecurity Certificate is an introductory credential. The quizzes offer unlimited attempts and the content is genuinely beginner-level. It is a starting point, not a destination. The Python module is worth revisiting with a supplemental resource like freeCodeCamp for more depth and hands-on practice before moving into security automation work.

---

## Documentation — the thing you build while building everything else

Documentation came up repeatedly throughout the CCNA coursework and the lab rebuild in the same week, which felt like a sign. The IOS commands `show version` and `show ip interface brief` pull most of the information needed for a device documentation record directly from the device. The time to run those commands is during commissioning, not during an outage.

The minimum viable device documentation record: device name, model, serial number, interface, MAC address, IP address, purchase date, in-service date, warranty expiration, and firmware version. Firmware versions should be consistent across all devices of the same type. Inconsistency makes troubleshooting harder and introduces unnecessary attack surface variance.

NetBox handles IPAM, asset tracking, and network topology documentation in the management VLAN as a living source of truth. The addressing list is a living document started from day one — not something reconstructed after the fact from memory.

---

## Certification status

The Google Cybersecurity Certificate is complete. It is a legitimate starting point for someone new to the field and the Python module has direct application to security automation work. It is not a viable standalone credential and the course structure makes this obvious — unlimited quiz attempts and beginner-level content throughout. The credential is checked, the concepts are logged, and the next target is Network+.

The CCNA summer course from NetworkChuck is running in parallel and the networking content is hitting significantly harder than anything in the Google course. The goal is to understand what the Network+ exam is testing well enough that the exam itself is a formality rather than a surprise — not to memorize answers but to have actually configured the things being tested. PNETLab in the homelab makes this possible.

---

## Retrospective

- **A controlled demolition is faster than sustained technical debt.** Four months of drift produced a lab that was harder to operate than a fresh build. Starting clean from documented baselines — even though it requires rebuilding everything — produces a more operable environment in less total time than attempting to fix accumulated inconsistencies in place.
- **Document the quirks, not just the working state.** The Vertiv GXT5 reporting 37% on bypass is exactly the kind of detail that causes a false shutdown event six months later. Correct documentation is not "NUT works" — it is "NUT works, bypass reads 37%, shutdown threshold must be set below 37%."
- **Install infrastructure touchpoints before you need them.** A Portainer agent and Falco instance installed during initial build cost minutes. The same work done as a retrofit costs hours and risks service interruption. Plan the full stack before building the first layer.

---

## Cheatsheets

### TrueNAS dataset setup

```bash
# Dataset type for application data directories
# pgdata, config directories → type: generic
# primary data directory    → type: apps (required for correct permissions)
```

### NUT — Vertiv GXT5 configuration

```bash
# Blacklist conflicting serial driver
echo "blacklist cdc_acm" | sudo tee /etc/modprobe.d/blacklist-cdc_acm.conf

# udev rule for USB permissions
cat <<'EOF' | sudo tee /etc/udev/rules.d/62-nut-usbups.rules
ATTR{idVendor}=="10af", ATTR{idProduct}=="1000", MODE="664", GROUP="nut"
EOF
sudo udevadm control --reload-rules && sudo udevadm trigger

# Install patched driver
git clone https://gitea.com/ftfy/nut-liebert-psa5
cd nut-liebert-psa5 && sudo ./install.sh

# /etc/nut/nut.conf
MODE=netserver

# /etc/nut/ups.conf
[vertiv-gxt5]
    driver = usbhid-ups
    port = auto
    vendorid = 10af
    productid = 1000
    usb_hid_rep_index = 1
    pollonly
    desc = "Vertiv GXT5 UPS"

# /etc/nut/upsd.conf
LISTEN 0.0.0.0 3493

# Start and enable
sudo systemctl enable --now nut-driver@vertiv-gxt5 nut-server

# Verify
upsc vertiv-gxt5
upsc vertiv-gxt5 ups.status
upsc vertiv-gxt5 battery.charge
upsc -l
sudo systemctl status nut-driver@vertiv-gxt5 nut-server
do clear ip nat translation *
```

### AdGuard Home

```bash
# Install (Debian/Ubuntu LXC)
sudo apt update && sudo apt upgrade -y && sudo apt install curl -y
curl -s -S -L \
  https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh \
  | sh -s -- -v
# Access setup UI at http://[LXC-IP]:3000 immediately after install
```

### Cisco IOS — Layer 2 troubleshooting

```text
show ip interface brief                      # interface inventory and status
show interfaces status                       # speed, duplex, connection state
show mac address-table                       # full CAM table
show mac address-table interface [port]      # CAM entries for a single port
show interfaces description                  # port descriptions
show arp                                     # IP-to-MAC mappings
show cdp neighbors                           # directly connected Cisco devices
show version                                 # hardware info, firmware, serial
clear mac address-table dynamic              # clear learned MAC entries
```

### Cisco IOS — routing and EIGRP

```text
show ip route                    # routing table (D = EIGRP learned)
router eigrp 1                   # enter EIGRP config, AS number 1
network 192.168.1.0              # advertise network, send hellos on matching interfaces
show ip eigrp neighbors          # verify adjacency formation
```

### Cisco IOS — NAT

```text
ip nat inside                                           # apply to LAN interface
ip nat outside                                          # apply to WAN interface
ip nat inside source list [acl] interface [int] overload  # configure PAT/overload
show ip nat translation                                 # verify active translations
do clear ip nat translation *                           # clear NAT table
```

### Cisco IOS — ACL

```text
access-list 10 permit 192.168.1.5                              # standard ACL
access-list 101 permit tcp 192.168.1.0 0.0.0.255 host 10.0.0.5 eq 80  # extended ACL
interface g0/0
  ip access-group 10 in                                        # apply to interface
show ip access-lists                                           # verify
```

### Cisco IOS — baseline config

```text
enable
configure terminal
hostname [device-name]
enable secret [password]
line vty 0 4
  password [password]
  login
interface [int]
  no shutdown
write memory
show version
show ip interface brief
```

### Subnetting math

```text
2^n - 2 = usable hosts          (n = host bits needed)
256 - 2^n = last octet of subnet mask
32 - n = CIDR notation

Bit chart: 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1
Increment = value of the rightmost network bit in decimal

Transfer time:
File (GB) × 8 = bits
Bits ÷ link speed (bps) = seconds
Seconds × 1.20 = realistic estimate with overhead
```

### Python basics

```python
# Data types
string_var = "192.168.1.1"          # string
int_var = 3493                       # integer
float_var = 37.5                     # float
bool_var = True                      # boolean
list_var = ["eth0", "eth1", "eth2"]  # list

# Conditional
if battery_charge <= 20:
    print("Low battery warning")

# Comparison operators
# ==  equal to
# !=  not equal to
# >   greater than
# <   less than
# >=  greater than or equal to
# <=  less than or equal to

# For loop
for interface in interfaces:
    print(f"Checking {interface}")

# Function with return
def check_status(service):
    if service == "running":
        return True
    return False

# type() — returns data type of any object
type("hello")    # <class 'str'>
type(3493)       # <class 'int'>
```
