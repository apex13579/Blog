# 📔 Homelab Overhaul: Building a Command & Control Stack
**Date:** April 6, 2026  
**Category:** Infrastructure / DevOps  

I’ve continued my homelab overhaul with a fresh base stack running on **Linux Mint**. The first VM I built is my **Command and Control (C2) node**, which acts as the central point for deploying, managing, and monitoring the rest of the environment. 

### 🖥️ Node Specifications
*   **Hypervisor:** Proxmox
*   **OS:** Linux Mint
*   **vCPU:** 4 Cores
*   **RAM:** 4GB (3814.697 MiB)
*   **Disk:** 40GB Virtual Disk
*   **Features:** QEMU Guest Agent Enabled

My design philosophy is simple: keep the node as lightweight as possible so it remains responsive even under load. That means containerizing nearly everything using Docker.

---

## 🏗️ The Stack Deployment

### 1. The Foundation: Docker
Installing `docker.io` provides the Docker Engine, CLI, and containerd. I verify that the Docker daemon is active and enabled so it starts automatically on reboot. Docker is the foundation for everything else in this node.

### 2. Orchestration: Portainer
Portainer is a lightweight management GUI that sits on top of Docker. 
*   **Persistence:** Created a volume `portainer_data` to ensure config survives updates.
*   **Access:** Bound port `9443` (HTTPS) and mounted the Docker socket to give Portainer direct access to the Docker API.

### 3. Credential Management: Vaultwarden
I deployed **Vaultwarden**, a lightweight Bitwarden-compatible manager written in Rust. It is extremely resource-efficient.
*   **SOP:** I use a long master passphrase and rotate passwords every 3–6 months. Good password hygiene is the easiest way to avoid data breaches.

### 4. Notifications: ntfy.sh
I use **ntfy** for push notifications regarding hardware or software issues. It’s mapped to port `80` for simple internal access.

---

## 🤖 Automation with Ansible
To handle automated nightly updates, I installed **Ansible** via the official PPA.

I created an Ansible playbook, `update_system.yml`, to:
1. Update all packages.
2. Clean the package cache.
3. Remove unused dependencies.

**Maintenance Window:** I configured a cron job via Ansible to run this playbook at **2:00 AM EST** daily, ensuring the system stays patched without manual intervention.

---

## 🎓 Progress Report
*   **Certifications:** Completed **2/9** of the Google Data Analytics course. Progressing steadily alongside the lab rebuild.

---

## 🛠️ Bash Cheatsheet: Commands Used

| Command | Purpose | Why it Matters |
| :--- | :--- | :--- |
| `sudo` | Super User Do | Grants admin/root privileges for system changes. |
| `apt update` | Refresh Repo | Syncs local package list with remote repositories. |
| `docker run` | Start Container | Deploys services with flags like `-d` (background) and `-p` (ports). |
| `timedatectl` | Time Mgmt | Ensures logs and cron jobs sync with the correct timezone. |
| `ansible ... -m cron` | Automation | Creates scheduled tasks for maintenance windows. |

---

> [!IMPORTANT]
> **Security Tip:** If you ever lose your Portainer admin password, stop the container and run the `helper-reset-password` utility using the `portainer_data` volume to generate a temporary login.
