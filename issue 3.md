# Home Lab Notes: Stabilizing the Stack
**Issue 3 — From Chaos to Clean Architecture**
**Date: May 2, 2026**

---

**Why I Do This on Old Hardware on Purpose**

I want to be upfront about something. My home lab runs on cheap, used hardware. That is not a mistake or a budget problem. It is the whole point.

In the IT world, people love to say "just throw more RAM at it" when something runs slow. And sure, if you work for a company where downtime costs money, that makes sense. But for someone like me who is just trying to learn, buying expensive hardware would actually get in the way. When you only have 4 or 8 gigs of RAM to work with, you are forced to actually understand what is eating it. You learn why a service is slow. You figure out what you actually need versus what just sounds cool.

My goal with this lab is to prove that you do not need a server rack or a big power bill to build something real. A $20 laptop from a thrift store can run a serious setup if you know what you are doing. And if you don't know what you are doing yet, well, that's what the lab is for.

---

**The Foundation: Why I Use Proxmox**

The most important decision I made early on was picking Proxmox VE as my virtualization platform. Here is the difference that matters: Proxmox is a Type-1 hypervisor, which means it runs directly on the hardware. There is no Windows or Mac OS sitting underneath it. Programs like VirtualBox or VMware Workstation are Type-2, meaning they run as apps inside another operating system. That adds overhead, which in my case I cannot afford.

Inside Proxmox, I set up what I call a Command Server. It is one virtual machine that handles everything management related: monitoring, organizing my containers, and remote access. The logic behind it is pretty simple. If I am running an experiment that crashes something, I still want to be able to log in and see what happened. Keeping the management stuff separate and giving it priority means I always have a way back in.

---

**3D Printing My Way to a Better Setup**

Professional cable management stuff is expensive. Like, stupidly expensive for what it is. So I started 3D printing my own.

This is not just about looks either. Bad cable routing in a small space traps hot air and causes hardware to overheat over time. I printed cable guides, a small patch panel, and some mounting brackets for my mini PCs. I use PETG filament for anything that sits near heat since it handles it better than regular PLA.

The money I save on enclosures and mounts goes toward things that actually matter, like faster SSDs or more RAM.

---

**The Day Everything Went Down**

A few weeks ago, Vaultwarden, ntfy, and Portainer all stopped responding at the same time. My first thought was that the containers had crashed. They had not.

The real problem was that two programs on the host machine, an old Nginx install and a CUPS print service, had started up automatically and grabbed the ports my Docker containers needed. In Linux, only one thing can use a port at a time. So when Docker tried to set up its routing, port 80 was already taken. The containers were running fine. They just had no way to receive traffic.

This was a good reminder that "the container is running" and "the container is reachable" are two completely different things.

---

**How I Fixed It (The Nuclear Reset)**

Restarting the containers did nothing, which makes sense in hindsight. The problem was not the containers. The problem was the network routing underneath them.

Here is what actually worked. First I stopped all the containers at once with `docker stop $(docker ps -aq)`. Then I used `sudo ss -tulpn` to find what was sitting on the ports I needed. Once I identified the culprits (Nginx and CUPS), I disabled them permanently with `systemctl disable --now`. After that, I cleared the network routing tables and restarted the Docker service completely so it could rebuild everything from scratch.

When I ran `curl -I` and got a response back, I knew it was fixed. The whole thing took about 45 minutes once I figured out what was actually wrong.

---

**IP Addresses: More Than Just Numbers**

One thing the Google Cybersecurity course hammered into me is that you cannot secure a network you do not understand. An IP address is not just a label. It is a 32-bit binary number that determines how traffic gets routed across a network.

In my lab I use the 10.x.x.x address range with a /24 subnet on each segment, which gives me 254 usable addresses per segment. More importantly, I split the lab into separate segments on purpose. Management tools live on one subnet, services on another, and anything experimental gets its own isolated segment. That way if something goes wrong in the sandbox, it cannot easily spread to the rest of the lab. This is basically a beginner version of "zero trust" networking, where nothing gets access just because it is connected.

---

**Switching to SQLite Changed Everything**

Before this, my containers were using PostgreSQL as their database backend. PostgreSQL is great if you have a lot of users hitting a database at the same time. I have one user, which is me.

A PostgreSQL container sitting idle uses around 200MB of RAM. SQLite, on the other hand, is just a file on disk. No background service, no network configuration, no extra moving parts. When I moved Vaultwarden and Gitea over to SQLite, my idle memory usage dropped by about 40% and the containers started up noticeably faster.

The lesson here is that "enterprise grade" does not automatically mean "right for your situation."

---

**Fixing My Notes Setup**

For a while I was running Joplin Server, which is a self-hosted syncing service for the Joplin note-taking app. It kept throwing SQLITE_CANTOPEN errors whenever the storage mount had even a brief delay. The ironic part was that I could not write down notes about fixing the problem because my note system was the thing that was broken.

The fix was switching to WebDAV, which is basically just syncing notes as plain files over HTTP. There is no database involved. If I want to back up my notes, I run a copy command. That is it. Getting rid of the complexity fixed the reliability completely.

---

**Python is the Glue**

As part of the Google Cybersecurity track I have been learning Python, and it has started showing up everywhere in my lab.

I wrote a script I call Sentinel that checks my Docker Compose files before I deploy anything. It looks for things like containers set to run in privileged mode, hardcoded passwords in environment variables, and image versions that are not pinned to a specific release. It is not fancy but it catches dumb mistakes before they become real problems. The idea is to automate the boring security checks so I can spend my attention on things that actually need thinking.

---

**Getting Alerts on My Phone**

A lab that breaks silently is not really a lab, it is just a pile of computers. So I set up ntfy, which is a lightweight notification service that works by sending simple HTTP requests. If a script detects something wrong, like a high CPU temperature or a failed SSH login, it sends a notification.

I also wrote a small Bash script that acts as a bridge between ntfy and Telegram. It checks for new alerts and forwards them to my phone as encrypted messages. The nice part is this does not require any open ports on my router. It is all outbound traffic.

---

**Where I Am at with Certifications**

The longer term goal is a T-shaped skill set, meaning broad general IT knowledge with a deeper focus on security. I finished the Google Data Analytics and Project Management certificates earlier, which helped a lot with understanding how to work with data and plan technical projects.

Right now I am about halfway through the Google Cybersecurity Professional certificate. After that the big one is the CCNA, which I am planning to tackle this summer alongside Network Chuck's course. The lab is already set up to support this with Proxmox configured to run GNS3 and Cisco Packet Tracer for network simulation work.

---

**What I Would Tell Myself Six Months Ago**

Write everything down. Even if it feels obvious in the moment, you will not remember it in three weeks. Keep services simple and separate so that when one breaks it does not take three others with it. Learn to use the command line early because almost everything worth doing happens there. And use 3D printing for physical stuff if you have access to a printer, it saves real money.

The stack is stable now. On to the next thing.

Stay curious. Stay secure.
