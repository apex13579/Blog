
title: "Building the Sovereign Stack: From Cable Hell to ZFS Pools"
date: 2026-06-03
category: Infrastructure & Networking
tags: [HomeLab, TrueNAS, Proxmox, LinuxMint, Network+]
---

## The Catalyst
I finally got my 3D printer parts in and went to town on my server rack. I printed custom cable management, keystone patch panels, and blanks. I installed my new UPS and cleaned up the rat's nest using velcro and bread ties. Then, disaster struck. I finished the physical cleanup, fired everything up, and my Proxmox node completely dropped off the network. Here is how I recovered my hypervisor, built a new storage array, and fought a brutal permission battle.

---

## Rack Overhaul and Layer 1 Gremlins
Cleaning the rack made the hardware look enterprise-grade, but it completely broke my environment. The Proxmox host went totally unreachable. I didn't change any network configurations, so I knew it was a physical layer issue. 

> **> DANGER:** Messing with Layer 1 cables without tracing your drops will kill your management plane. Always label both ends of every run before you disconnect anything.

I resisted the urge to panic-reconfigure the network interface files. I left the host online and let the switch ARP and DHCP tables cycle. After a few minutes, the IP self-corrected and management access restored itself. 

With the hypervisor stable, I spun up my primary DNS VM. I tried to use Alpine Linux to keep the footprint tiny. After an hour and a half of wrestling with configuration files, I rage quit. I went back to Linux Mint. 

> **> SUCCESS:** Linux Mint trades a slightly larger RAM footprint for immediate, out-of-the-box driver stability. It kept the deployment moving when Alpine stalled my entire night.

This VM now hosts AdGuard Home for network-wide ad blocking and Network UPS Tools (NUT) to safely manage power downs.

---

## TrueNAS SCALE and the Nextcloud Permission Trap
I initialized TrueNAS SCALE on my second Dell PowerEdge R610. I set up a storage pool using RAIDZ2. It gives me two-disk fault tolerance. The array is small because hard drive prices are brutal right now, but it works. 

I mapped the share to the network and tried to spin up Nextcloud and Immich. Nextcloud failed instantly due to a massive permissions error. The web app couldn't write data to the ZFS mount point. I dropped into the TrueNAS shell and forced the system to give the web daemon recursive ownership.

`TrueNAS Shell`
```bash
# Elevate to root privileges
sudo su

# Force web daemon ownership across the entire dataset path
chown -R www-data:www-data /mnt/example/*
```

This fixed the storage layer, but the Nextcloud web UI immediately threw a domain trust error. I used `nano` to edit the config file. I finally reached the login screen, but the app still refused my credentials due to persistent database bugs. 

> **> WARN:** Nextcloud's permission implementation on TrueNAS SCALE has been a known, documented community issue for 2.5 years. Do not waste days fighting it. I yanked the container completely and am replacing it with lightweight, decoupled apps.

---

## Cisco IOS Baselines and the OSI Model
I am prepped to flash OPNsense onto my Cisco ASA 5512-X firewall. To configure it, I hooked up a USB-to-serial rollover cable to the console port. I fired up my Linux terminal and set the serial connection speed to 9600 baud. 

My strategy for this security edge is strict: I am deploying a zero-trust, default-deny ruleset. I will open ports one by one as things break, rather than trying to map a complex policy beforehand. 

While labbing, I mapped out exactly how data frames wrap layers when moving down the OSI stack:

`OSI Data Frame Layer Architecture`
```text
[L2 trailer] [DATA] [L4 header] [L3 header] [L2 header]
```

Every hop through a router strips the Layer 2 source address and replaces it with its own. Before any data moves, my PC initiates a TCP 3-way handshake (`SYN` -> `SYN-ACK` -> `ACK`) to verify the path is completely open.

---

## Progress and Skill Overload
I am shifting my configuration habits from basic GUI point-and-clicking to rigid command-line baselines. According to my Home Lab DB tracking, my physical rack cleanup reduced unlabelled links by 100%. However, undocumented cable swaps cost me an extra 45 minutes of troubleshooting time during the Proxmox outage. I am implementing a strict 6-task-a-day study framework over the weekends to keep my execution speed high.

---

## Retrospective
* **Label everything:** Use different colorways for your patch cables. If you don't map both ends, Layer 1 will bite you during a physical migration.
* **Ditch the monoliths:** When an application like Nextcloud fights your storage permissions for three hours, drop it. Lightweight, single-purpose containers are much easier to manage.
* **Stop relying on active lookups:** I waste too much time googling basic commands inside my VMs. I need to run through my Anki flashcards daily to build raw command-line muscle memory.

---

## Certification Status
I am halfway through the Google Cybersecurity Certificate, but the SQL modules are incredibly dry and the instructor lacks energy. To keep my momentum, I started Week 2 of Network Chuck’s Free CCNA course. The networking concepts are hitting home much faster. Once the Google cert is wrapped, I am taking a step back to target the ISC2 CC exam to build up my portfolio before moving to standard security certs.

---

## The Cheatsheets

### Bash
`bash_cheatsheet.sh`
```bash
# Open the terminal text editor inside your shell environment
nano filename.txt

# Trace the exact route and IP hops to a target domain without resolving DNS names
tracert -d cisco.com
```

### Networking
`cisco_ios_baseline.txt`
```text
# Standard Cisco IOS initial configuration workflow
enable                  # Enter privileged EXEC mode
configure terminal      # Drop into global configuration mode
hostname Switch01       # Apply specific device naming convention
write erase             # Completely wipe old startup configs on used gear
```

`interface_management.txt`
```text
# Basic port auditing and status verification
show ip interface brief # Output a clean snapshot of all interface states
show running-config     # Verify active running parameters against your template
interface f0/1          # Enter interface configuration mode for FastEthernet 0/1
shutdown                # Disable the port completely
no shutdown             # Re-enable the interface and bring the link up
```

`subnet_math_reference.txt`
```text
# Classless Inter-Domain Routing (CIDR) /24 standard boundary
A standard /24 subnet mask = 255.255.255.0
Total Address Space        = 256 addresses
Network Identifier         = -1 address
Broadcast Address          = -1 address
Total Usable Hosts         = 254 reachable hosts
```
