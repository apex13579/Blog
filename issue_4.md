Core Objective: Establish a robust, self-hosted "Sovereign" infrastructure by deploying Type-1 hypervisors and resilient storage, alongside advancing foundational networking (transitioning to Network+) and security principles. The overarching goal is shifting from proprietary dependencies to controlled, open-source or heavily managed environments.

Specific Linux Commands & Syntax:

nano (terminal text editor)

tracert -d [domain] (Windows route tracing)

Cisco IOS: show ip interface brief, show running config, configure terminal (or conf t), interface fastethernet 0/1, shutdown / no shutdown, write erase, enable

Linux Permissions: sudo su, chown -R www-data:www-data /mnt/example/*

Tool Versions & Software: Packet Tracer, Alpine Linux (abandoned), Linux Mint, Home Assistant, Matter Bridge, MQTT, Proxmox, TrueNAS SCALE, Nextcloud, Immich, AdGuard Home, Network UPS Tools (NUT), Jellyfin, Handbrake.

Hardware Specs: Dell R610 (TrueNAS pool in RAIDZ2), Cisco ASA 5512-X (targeted for OPNsense flashing), custom 3D printed components for rack management, serial rollover cable + USB adapter.

Friction Points & Resolutions:

Friction: Alpine Linux proved too time-intensive for the current deployment sprint. Resolution: Pivoted to Linux Mint for rapid deployment of host OS tasks.

Friction: Physical rack rewiring caused Proxmox network unreachable state. Resolution: Allowed the switch ARP/DHCP tables to cycle; the IP self-corrected.

Friction: Nextcloud deployment on TrueNAS failed due to www-data ownership conflicts and domain trust bugs. Resolution: Applied recursive chown to correct directory permissions, but ultimately deprecated the Nextcloud container entirely in favor of researching lightweight, decoupled alternatives due to persistent legacy bugs.

Building the Sovereign Stack: From Infrastructure Foundations to Immutable Systems
Date: May 2026
Category: Infrastructure & Networking
Tags: HomeLab, TrueNAS, OPNsense, Proxmox, LinuxMint, Network+

THE CHALLENGE
The modern digital landscape requires a shift away from fragile, vendor-locked architectures toward resilient, self-hosted environments. This month's deployment cycle focused on auditing and rebuilding a sovereign home data center, replacing black-box proprietary hardware with open-source firmware, and establishing a zero-trust network baseline. The objective was to solidify foundational networking protocols while standing up scalable, containerized services that lay the groundwork for truly immutable infrastructure.

THE SOVEREIGN STACK
The physical and logical topology relies on enterprise-grade hardware repurposed for sovereign control:

Compute & Virtualization: Proxmox VE hosting diverse Linux Mint VMs and LXC containers.

Storage: Dell PowerEdge R610 running TrueNAS configured with a RAIDZ2 pool for high-fault-tolerance data redundancy.

Edge Security: Cisco ASA 5512-X, currently being prepped to flash OPNsense to enforce a default-deny ruleset.

Core Services: Home Assistant (with MQTT and Matter Bridge), AdGuard Home, Network UPS Tools (NUT), and Jellyfin.

Physical Layer: Custom 3D-printed keystone patch panels and cable management arrays.

TECHNICAL IMPLEMENTATION
1. Network Device Provisioning & Baselines
Establishing a standardized configuration methodology is critical for network maintainability. Before moving to GUI-based firewall rules on OPNsense, foundational switch and router hygiene was enforced via standard serial console (9600 baud).

A standard deployment template was utilized to baseline legacy Cisco hardware before transition:

Plaintext
1. Hostname application
2. Banner MOTD
3. Enable secret
4. Console password and login
5. VTY password and remote access settings
6. Service password-encryption
7. Management IP on VLAN 1
8. Port descriptions
9. Saving the configuration
Note: When decommissioning or repurposing legacy gear, issuing write erase from privileged exec mode (enable) ensures no orphaned configurations introduce security vulnerabilities into the new topology.

2. Storage Architecture and Permission Auditing
A TrueNAS instance was initialized on the Dell R610, establishing a RAIDZ2 ZFS pool. During the deployment of web-facing applications, specifically Nextcloud, a critical permission mismatch occurred preventing the application container from writing to the mounted dataset.

To resolve the data-layer conflict, ownership was forcefully reassigned to the web daemon user via the TrueNAS shell:

Bash
sudo su
chown -R www-data:www-data /mnt/example/*
Why this matters: Containerized applications operating without root privileges (a security best practice) require explicit ownership of their mapped volume mounts. Failing to align Host OS permissions with Container UID/GIDs results in silent failures or domain trust errors.

3. The Hypervisor and Services Layer
Proxmox serves as the core hypervisor. To expedite deployment, Linux Mint was selected as the standard host OS for VMs over Alpine Linux, trading a marginally larger footprint for rapid deployment and out-of-the-box driver compatibility. Core infrastructure services, such as the primary DNS sinkhole (AdGuard Home) and UPS management (NUT), were decoupled and isolated within these VMs to prevent single points of failure.

THE WIN
The lab infrastructure has successfully transitioned from an unmanaged, flat topology to a structured, sovereign stack. By standardizing the physical layer with custom 3D-printed management and isolating core services across TrueNAS and Proxmox, the environment is now stable enough to support advanced cybersecurity deployments. This structural integrity directly supports the transition toward declarative, immutable infrastructure, where state is defined by code rather than manual GUI configurations.

LESSONS LEARNED
OSI Layer 1 Integrity: Physical topology alterations (rewiring the server rack) temporarily severed Proxmox connectivity. Strict adherence to cable labeling conventions and waiting for ARP cache expiration is vital before attempting logical troubleshooting.

Sunk Cost Fallacy in Software: Persistent permission bugs and domain trust issues within monolithic applications like Nextcloud can drain engineering cycles. Deprecating a failing monolithic service in favor of modular, lightweight applications is often the superior architectural decision.

Zero-Trust from Day Zero: When bringing new edge devices online (like the planned OPNsense deployment), starting with a default-deny, zero-trust ruleset and opening ports selectively is far more secure than attempting to map all required traffic prior to deployment.
