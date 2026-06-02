Building the sovereign stack: from cable hell to ZFS pools
June 3, 2026 · Infrastructure & Networking · HomeLab, TrueNAS, Proxmox, Linux Mint, Network+
The catalyst
My 3D printer parts finally arrived and I went to town on the server rack. I printed custom cable management clips, keystone patch panels, and blanks. I installed a new UPS, cleaned up the rat's nest with velcro and zip ties, and stepped back feeling good about life. Then disaster struck: I fired everything up and my Proxmox node completely dropped off the network. Here's how I recovered the hypervisor, built a new storage array, and fought a brutal permissions battle along the way.

Rack overhaul and layer 1 gremlins
The cleanup made the hardware look enterprise-grade — and completely broke my environment. The Proxmox host went totally unreachable. I hadn't changed any network configs, so I knew immediately this was a physical layer problem.

⚠ Danger: Touching Layer 1 cables without tracing your drops first will kill your management plane. Label both ends of every run before you disconnect anything.
I resisted the urge to panic-reconfigure network interface files, left the host online, and let the switch's ARP and DHCP tables cycle. A few minutes later the IP self-corrected and management access came back. Crisis averted by doing nothing.

With the hypervisor stable, I spun up my primary DNS VM. I tried Alpine Linux first to keep the footprint tiny — and spent an hour and a half wrestling config files before rage-quitting back to Linux Mint.

✓ Success: Linux Mint trades a slightly larger RAM footprint for immediate, out-of-the-box driver stability. It kept the deployment moving when Alpine stalled my entire night.
This VM now hosts AdGuard Home for network-wide ad blocking and Network UPS Tools (NUT) for safe power-down management.

TrueNAS SCALE and the Nextcloud permission trap
I initialized TrueNAS SCALE on my second Dell PowerEdge R610 and set up a storage pool using RAIDZ2 — a ZFS configuration that tolerates two simultaneous disk failures without data loss. Hard drive prices are brutal right now so the array is small, but it works.

I mapped the share to the network and tried to spin up Nextcloud and Immich. Nextcloud failed instantly with a massive permissions error: the web daemon couldn't write to the ZFS mount point. I dropped into the TrueNAS shell and forced recursive ownership:

TrueNAS shell
sudo su
chown -R www-data:www-data /mnt/example/*
That fixed the storage layer, but the Nextcloud web UI immediately threw a domain trust error. I edited the config with nano, reached the login screen, and then hit persistent database bugs blocking authentication. I finally pulled the plug on the whole container.

⚠ Warning: Nextcloud's permission implementation on TrueNAS SCALE has been a known, documented community issue for over two years. Don't waste days fighting it. Replace it with lightweight, decoupled apps instead.
Cisco IOS baselines and the OSI model
I'm prepping to flash OPNsense onto a Cisco ASA 5512-X. To get into the device, I hooked a USB-to-serial rollover cable to the console port and connected at 9600 baud from my Linux terminal.

My security strategy is strict: zero-trust, default-deny, open ports one by one as things break rather than trying to map a complex policy upfront.

While working through the concepts, I mapped out how data is encapsulated moving down the OSI stack. Each layer wraps the payload from the layer above it:

OSI encapsulation order (outermost → innermost)
[L2 header] [L3 header] [L4 header] [DATA] [L2 trailer]
At every router hop, the Layer 2 header is stripped and rewritten with the next-hop MAC address — the Layer 3 IP address stays intact end-to-end. Before any data moves, TCP runs a three-way handshake (SYN → SYN-ACK → ACK) to confirm the path is open.

Progress and skill overload
I'm shifting away from GUI point-and-click toward rigid command-line baselines. The rack cleanup reduced unlabelled links by 100% according to my home lab DB, but undocumented cable swaps during the cleanup cost me 45 extra minutes of troubleshooting. Lesson logged. I'm also implementing a strict 6-task-a-day study framework on weekends to keep execution speed high.

Retrospective
Label everything. Use different colors for your patch cables. If you don't map both ends before you pull anything, Layer 1 will bite you every single time.
Ditch the monoliths. When an app fights your storage permissions for three hours straight, drop it. Lightweight, single-purpose containers are dramatically easier to manage and debug.
Build CLI muscle memory offline. Googling basic commands inside a production VM is slow and embarrassing. Run Anki flashcards daily so the commands are there when you need them.
Certification status
I'm halfway through the Google Cybersecurity Certificate, but the SQL modules are dry and the instructor lacks energy. To stay sharp, I started Week 2 of NetworkChuck's free CCNA course — the networking content is hitting home much faster. Once the Google cert is wrapped, I'll step back and target the ISC2 CC exam to build out my portfolio before moving into standard security certs.

The cheatsheets
Bash
bash_cheatsheet.sh
# Open the terminal text editor
nano filename.txt

# Trace the route and IP hops to a target (no DNS resolution)
tracert -d cisco.com        # Windows
traceroute -n cisco.com     # Linux
Networking
cisco_ios_baseline.txt
enable                  # Enter privileged EXEC mode
configure terminal      # Drop into global configuration mode
hostname Switch01       # Apply device naming convention
write erase             # Wipe startup config on used gear
interface_management.txt
show ip interface brief # Snapshot of all interface states
show running-config     # Verify active config against your baseline
interface f0/1          # Enter config mode for FastEthernet 0/1
shutdown                # Disable the port
no shutdown             # Re-enable and bring the link up
subnet_math_reference.txt
# CIDR /24 subnet breakdown
Subnet mask        = 255.255.255.0
Total address space = 256 addresses
Network address    = -1
Broadcast address  = -1
Usable hosts       = 254
