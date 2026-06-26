---
title: "Burning it down to build it right: lab rebuild, UPS wars, and the summer of CCNA"
date: 2026-07-01
category: Infrastructure & Networking
tags: [HomeLab, TrueNAS, Proxmox, NUT, AdGuard, Immich, CCNA, Network+, Python, VLANs, NAT, Subnetting]
---

## The catalyst

Four months into this lab, I had to have an honest conversation with myself. The vision had
grown faster than the architecture could keep up with. Services were scattered across machines
without a clean rationale, the network was still flat in places it had no business being flat,
and half of what was running existed because it worked once, not because it belonged there. The
only move that made sense was a controlled demolition — blow up the old stack, document
everything on the way back up, and use the rebuild as a forcing function to wire the CCNA and
Google Cybersecurity coursework into something physical and real. This is that story.

---

## Blowing it up

Tearing down a lab you have been running for months is genuinely uncomfortable. Every service
that goes dark is something you have to rebuild from scratch, reconfigure from memory, and
document properly this time. I have come to believe that discomfort is the point. A lab you
cannot destroy and rebuild cleanly from documentation is not a lab — it is technical debt with
a power strip.

The rebuild sequence was deliberate. NAS apps first, then LXCs, then VMs, then standalone
devices. Storage is the foundation everything else depends on, so it goes in first. Services
that talk to storage go in second. Everything else follows in dependency order. Skipping that
sequence means spinning up services that cannot connect to the things they need and spending
two hours troubleshooting a problem that was actually just bad sequencing.

---

## The storage layer — TrueNAS apps first

Both Dell PowerEdge R610s got wiped and reconfigured from scratch. This time, rather than
running services as Proxmox LXCs competing for hypervisor resources, everything NAS-resident
runs as a TrueNAS SCALE native app. The storage layer has its own failure domain, completely
separate from the compute layer. That separation matters when something breaks at two in the
morning.

The first app up was Immich for photo backup. The real motivation: getting years of family
photos off Apple's infrastructure and onto hardware I physically control. If you have not done
this yet, do it. Dataset structure is what trips most people up on the first install.

```text
applications/
└── immich/
    ├── pgdata/     # type: generic — PostgreSQL data
    └── data/       # type: apps — required for correct permissions
```

> **Danger:** The `data` dataset must be set to type `apps`, not `generic`. This is a
> TrueNAS-specific permission requirement. Set it wrong and Immich silently fails to
> initialize — and the logs will point you somewhere completely unhelpful.

After Immich came Uptime Kuma for service monitoring, which I populated with a running device
list immediately on install rather than leaving it empty to fill in later. Then Tailscale for
remote access. I refuse to open ports on my network. Tailscale handles NAT traversal
automatically, which means getting SSH access to any machine on the network from anywhere in
the world requires exactly zero firewall rule changes on my end.

I also evaluated WebDAV as a sync gateway for Joplin — my wife and I both use it for notes —
but killed the idea after the app gave me no way to set credentials through the interface. That
one goes on the list for when Nextcloud is running.

> **Win:** Every time a new service goes up, I add its IP and port to the addressing list
> before I do anything else. Takes thirty seconds. Has saved hours. Do it at install time,
> every time, without exception.

---

## The NUT war

This was the longest battle of the entire rebuild, and the one that deserves the most
documentation for anyone unlucky enough to own a Vertiv GXT5.

The GXT5 uses USB HID for UPS monitoring, which sounds simple enough. The problem is that its
HID data lives on interface 1, not the default interface 0 that every standard NUT driver
targets. Every driver connected cleanly, reported success, and returned nothing useful — because
it was talking to the wrong interface the entire time.

My first attempt ran NUT inside an Alpine LXC. I got the cgroup2 device rules configured, the
USB device appeared to pass through correctly, and the LXC still could not enumerate it
reliably. After hours of chasing this I made a decision that I should have made earlier: move
to a Linux Mint VM with USB passthrough configured through the Proxmox GUI.

> **Win:** VMs handle USB passthrough more reliably than LXCs because they present a full
> virtual USB controller to the guest. If a service needs direct hardware access — USB,
> serial, anything physical — use a VM. Save LXCs for network services and pure software.

The actual fix came from a patched NUT 2.8.2 build on Gitea (`ftfy/nut-liebert-psa5`). Two
config additions did the work: `usb_hid_rep_index = 1` to target the correct interface, and
`pollonly` to work around interrupt endpoint conflicts that caused the standard driver to hang
indefinitely. Configuration details are in the cheatsheet at the bottom.

One more thing worth logging that I will absolutely forget in six months: the GXT5 reports 37%
battery charge when running on bypass. If your shutdown threshold sits above 37%, every bypass
event triggers a false low-battery shutdown. Set it below 37% and save yourself the midnight
panic.

---

## AdGuard Home and the rest of the stack

AdGuard Home replaced Pi-hole this round. The cleaner interface, per-client statistics, and
native DoH/DoT upstream support made it the obvious move. The LXC runs Debian 12 — Debian 13
images for Proxmox have some reported rough edges in the community and I am not interested in
finding out firsthand this early in a fresh build.

One thing to catch during LXC creation: AdGuard needs passthrough to a physical network
interface. Without it the service starts, looks healthy, and silently fails to intercept any
DNS queries. It is a five-second checkbox in the Proxmox LXC config that is easy to miss and
extremely confusing to diagnose after the fact.

On top of AdGuard, two more pieces went in during this phase. A Portainer agent got installed
on the NUT VM in preparation for a Portainer instance that does not exist yet. And Falco went
directly onto the Proxmox hypervisor itself rather than inside a guest. 

That last decision matters: Falco at the hypervisor level watches the kernel syscall stream
for every guest simultaneously. An instance inside a single VM or LXC only sees that one
guest's activity. When your SIEM comes online, you want the full picture, not a fragment.

The principle behind both is the same — install infrastructure touchpoints during the initial
build while everything is already being touched. Retrofitting them later costs significantly
more time and often forces restarts that could have been avoided entirely.

---

## VLANs — security through constraint

With the compute and storage layers rebuilt, the network got the same treatment. The VLAN
architecture that came out of this is intentionally tight. Every VLAN maps to one broadcast
domain and one IP subnet, sized as small as possible for its realistic device count.

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

The tight subnets are not accidental — minimal blast radius if any one VLAN gets compromised.
The management VLAN gets a /27 because it carries the most devices. Everything else gets sized
to what it actually needs. This is not academic subnetting. It is security through constraint.

A VLAN is a broadcast domain. A router stops broadcasts, which means a router is required
before any traffic can cross VLAN boundaries. That boundary is the enforcement mechanism that
makes segmentation meaningful rather than decorative. According to 802.1Q there can be up to
4096 VLANs on a single network, with trunk ports carrying tagged frames between switches to
keep each VLAN's traffic correctly separated in transit.

---

## What the lab taught the coursework

Something interesting happened during the rebuild: the CCNA content I had been grinding
through started clicking in ways it had not before, because the lab was now a live version of
what the courses were describing. Here is what landed hardest.

**MAC addresses and Layer 2 troubleshooting**

A MAC address is a 12-character hexadecimal number. The first six characters are the OUI —
the Organizationally Unique Identifier — which maps directly to the manufacturer. The second
six uniquely identify the specific device. When you find an unknown device on your network, a
public OUI lookup tells you immediately whether you are looking at a Dell server, a Ubiquiti
WAP, or a Canon printer that has absolutely no business being on the management VLAN.

Switches live at Layer 2 and forward frames based on MAC addresses, not IP addresses. The
CAM table is where a switch stores everything it has learned about which MAC lives on which
port. Following a device across a switched network means tracing that table hop by hop until
you hit the endpoint. If you see multiple MAC addresses on a single port, you are almost
certainly looking at a downstream switch — one interface representing an entire world of
devices behind it.

ARP is the bridge between Layer 3 and Layer 2. Before a device can send a packet, it needs
to know the destination MAC address. If it does not have it cached, it broadcasts a request to
all-F destination MAC asking who owns the target IP. Only after that resolves can the frame
be properly addressed and forwarded. This is why a ping cannot proceed until ARP has done
its job first.

**Routing, EIGRP, and why routers matter**

Routers have two jobs: connecting networks and stopping broadcasts. That second job is the
one that makes network segmentation actually work. EIGRP — Enhanced Interior Gateway Routing
Protocol — lets routers share their routing tables automatically with neighbors in the same
autonomous system, so you are not manually configuring every route on every device. When two
EIGRP routers recognize each other and start exchanging topology information, that relationship
is called an adjacency. Routes learned through it show up in the table marked with `D`.

The routing table is the brain of the router. It is also where you will spend most of your
troubleshooting time. A route entry like `D 192.168.3.0/24 [90/5376] via 10.0.0.1` tells you
everything: learned via EIGRP, administrative distance of 90 (lower is more trusted, like
golf), metric of 5376, next hop at 10.0.0.1.

**NAT — the thing that makes private addresses work on the internet**

NAT translates private internal addresses to public external addresses so your devices can
reach the internet without exposing every machine directly. The four terms that end the
confusion permanently:

- Inside local: your device's private IP — you own it, it is private
- Inside global: your device's public IP — you own it, it is public  
- Outside global: the server's public IP — they own it, it is public
- Outside local: the server's address as seen inside your network — they own it, it is private

PAT, also called NAT overload, maps many private addresses to a single public address using
port numbers to track individual connections. This is what almost every home and small business
network runs. If internal devices can reach each other but not the internet, and the router can
ping the ISP, check NAT before anything else. I learned this the hard way on a practice lab
that took me much longer to debug than it should have.

**ACLs — filtering with precision**

Standard ACLs match on source IP only, run numbered 1-99, and apply close to the destination.
Extended ACLs match on source, destination, protocol, and port, run numbered 100-199, and
apply close to the source. Wildcard masks are the inverse of subnet masks — flip the bits.
ACLs process top-down, stop at the first match, and carry an implicit deny-all at the end
that will silently drop everything you forgot to permit. An ACL does nothing until it is
applied to an interface with a direction. That is the step most commonly forgotten.

**Subnetting — finally mechanical**

Subnetting stopped being an exercise and started being a tool the moment it connected to the
VLAN table above. Three steps, every time:

1. Determine the host count needed, convert to binary, count the bits required
2. Reserve those bits in the mask from the right, find the increment (the rightmost network
   bit back in decimal)
3. Add the increment repeatedly to find each successive network range

Win (Tony Norrell gets full credit for this one): Subnetting finally became mechanical the moment a good friend explained it as a pizza. /24 is 256 slices. Every time the CIDR goes up by 1, cut it in half. Every time it drops by 1, double it. /25 = 128. /23 = 512. No chart, no memorization, no stress. I owe Tony a coffee — or several. That analogy alone saved me more time than I can calculate.

**Python for security automation**

The final module of the Google Cybersecurity Certificate introduced Python as a security
automation tool, which is where the course finally became interesting. Strings for IP addresses
and log entries, integers for port numbers and counts, booleans for yes/no decisions, lists
for device inventories and blocklists. Conditional statements for automated decision-making,
loops for anything repetitive across device lists or log files. Return statements for passing
data back out of functions. These are not abstract concepts — they are the building blocks of
every script that makes a lab run itself instead of requiring manual intervention.

---

## Certification status

The Google Cybersecurity Certificate is finished. It is a legitimate starting point and
nothing more — unlimited quiz attempts, genuinely beginner-level content, and a credential
that opens doors primarily as a signal that you can finish something. The Python module was
the most directly applicable piece. The SQL modules were a slog. I am glad to have it done.

The next target is Network+. The CCNA summer course from NetworkChuck is running in parallel
and the networking content has been hitting significantly harder than anything in the Google
course. The goal is to understand what is being tested well enough that the exam itself is
a formality — not to memorize answers, but to have actually configured the things the
questions are asking about. Having a live lab where every concept has a physical counterpart
is the advantage I am building toward.

---

## Retrospective

- **A controlled demolition beats sustained technical debt.** Four months of drift produced
  a lab that was harder to operate than a fresh build. Rebuilding from scratch with documented
  baselines produced a more operable environment faster than trying to fix accumulated
  inconsistencies in place would have.
- **Document the quirks, not just the working state.** The GXT5 reading 37% on bypass is
  exactly the kind of detail that causes a false shutdown six months from now. "NUT works" is
  not documentation. "NUT works, bypass reads 37%, threshold must be set below 37%" is.
- **Install before you need it.** A Portainer agent and a Falco instance during the initial
  build take minutes. The same work done as a retrofit takes hours and risks service
  interruption. Plan the full stack before building the first layer.

---

## Cheatsheets

### TrueNAS — Immich dataset setup

```bash
# Dataset types for Immich
# pgdata → type: generic
# data   → type: apps  (required — do not skip this)
```

### NUT — Vertiv GXT5

```bash
# Blacklist conflicting serial driver
echo "blacklist cdc_acm" | sudo tee /etc/modprobe.d/blacklist-cdc_acm.conf
# Reboot after this step

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

# Start and enable services
sudo systemctl enable --now nut-driver@vertiv-gxt5 nut-server

# Verify
upsc vertiv-gxt5
upsc vertiv-gxt5 ups.status
upsc vertiv-gxt5 battery.charge
upsc -l
sudo systemctl status nut-driver@vertiv-gxt5 nut-server
```

### AdGuard Home

```bash
sudo apt update && sudo apt upgrade -y && sudo apt install curl -y
curl -s -S -L \
  https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh \
  | sh -s -- -v
# Access setup UI at http://[LXC-IP]:3000 immediately after install
# Service binds to port 53 after initial setup — do not wait
```

### Cisco IOS — Layer 2 troubleshooting

```text
show ip interface brief                       # interface inventory and status
show interfaces status                        # speed, duplex, connection state
show mac address-table                        # full CAM table
show mac address-table interface [port]       # CAM entries for a single port
show interfaces description                   # port descriptions
show arp                                      # IP-to-MAC mappings
show cdp neighbors                            # directly connected Cisco devices
show version                                  # hardware, firmware, serial number
clear mac address-table dynamic               # clear learned MAC entries
```

### Cisco IOS — routing and EIGRP

```text
show ip route                     # routing table (D = EIGRP learned)
router eigrp 1                    # enter EIGRP config, AS number 1
network 192.168.1.0               # advertise network, enable hellos on matching interfaces
show ip eigrp neighbors           # verify adjacency formation
```

### Cisco IOS — NAT

```text
ip nat inside                                            # apply to LAN-facing interface
ip nat outside                                           # apply to WAN-facing interface
ip nat inside source list [acl] interface [int] overload # configure PAT/overload
show ip nat translation                                  # verify active translations
do clear ip nat translation *                            # clear NAT table before reconfiguring
```

### Cisco IOS — ACL

```text
access-list 10 permit 192.168.1.5                                       # standard ACL
access-list 101 permit tcp 192.168.1.0 0.0.0.255 host 10.0.0.5 eq 80   # extended ACL
interface g0/0
  ip access-group 10 in                                                  # apply inbound
show ip access-lists                                                      # verify
```

### Cisco IOS — device baseline

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

### Subnetting

```text
2^n - 2 = usable hosts          (n = host bits needed)
256 - 2^n = last octet of subnet mask
32 - n = CIDR notation

Bit chart (right to left): 1 | 2 | 4 | 8 | 16 | 32 | 64 | 128
Increment = value of the rightmost network bit converted back to decimal

Transfer time estimate:
File size (GB) × 8 = total bits
Total bits ÷ link speed (bps) = seconds
Seconds × 1.20 = realistic estimate with 20% overhead
```

### Python basics

```python
# Data types
ip = "192.168.1.1"               # string
port = 3493                       # integer
charge = 37.5                     # float
is_online = True                  # boolean
interfaces = ["eth0", "eth1"]     # list

# Conditional statement
if battery_charge <= 20:
    print("Low battery — check UPS")

# Comparison operators
# ==  equal to        !=  not equal to
# >   greater than    <   less than
# >=  greater than or equal to
# <=  less than or equal to

# For loop
for interface in interfaces:
    print(f"Checking {interface}")

# Function with return statement
def is_running(service):
    if service == "active":
        return True
    return False

# type() returns the data type of any object
type("hello")    # <class 'str'>
type(3493)       # <class 'int'>
```
