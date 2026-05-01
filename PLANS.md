# 🏗️ Infrastructure & Deployment Roadmap

## 🛰️ Software Stack

### **Hypervisor & Runtime Security**
*   **Hypervisor:** ~~Proxmox~~ (Active)
*   **Runtime Security:** **Falco** (Standardizing across all nodes for threat detection)

### **Virtual Machines (VMs)**
*   **DB-VM:** ~~Vaultwarden, WebDAV, Portainer Agent, Gitea, Docker~~ | **Pending:** Falco
*   **CMD-VM:** ~~Flame, Ansible, Python, Ntfy.sh, Traefik, Portainer~~ | **Pending:** Falco, Pangolin
*   **DNS-PRIMARY:** AdGuard Home, NUT, Falco, Portainer Agent
*   **DNS-SECONDARY:** AdGuard Home, NUT, Falco, Portainer Agent
*   **PKI (Internal Trust):** Step-CA, Falco, Portainer Agent
*   **DATA-VAULT:** NFS, Samba, SQLite storage, Falco, Portainer Agent
*   **GUARD-DOG (Scanning):** ClamAV daemon + REST API, Falco, Portainer Agent
*   **SIEM:** S.I.B (SIEM in a Box), OpenVAS, FalcoSidekick, Portainer Agent
*   **HOMEAUTO:** Home Assistant Core, Mosquitto (MQTT), Falco, Portainer Agent
*   **HONEYPOT:** T-Pot, Falco, Portainer Agent

### **Linux Containers (LXC)**
*   **Jellyfin:** Media streaming
*   **Vaultwarden:** (Credential Migration)
*   **Gitea:** (Repository Migration)

### **Off-Site / Cloud (VPS)**
*   **Pangolin:** (Secure Tunneling/VPN)
*   **Falco:** Remote runtime monitoring

### **Network Attached Storage (NAS)**
*   **OpenMediaVault (OMV)**
*   **Falco:** Filesystem and access monitoring

---

## 🔌 Hardware Roadmap

### **Compute & Networking**
*   **NUC Cluster:** 3x Low-power nodes (Perfect for DNS/PKI high availability)
*   **Storage Units:** 2x UNAS PRO (Mass storage and Data-Vault)
*   **Switching:** ASUS GX10
*   **In-Progress:** Dell OptiPlex 5040 (AI/Research Node)

---

> [!TIP]
> **Security Insight:** By deploying **FalcoSidekick** with your SIEM, you can pipe real-time alerts from your containers and VMs directly into your SIEM dashboard. This is a massive "Sovereign" play for documenting incident response.
