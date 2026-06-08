### 🌐 Proxmox VE (PVE) Hypervisor Management

#### 🖥️ Virtual Machine Management (`qm`)

| Command | What It Does | Why it Matters |
| :--- | :--- | :--- |
| `qm list` | Lists all Virtual Machines configured on the node along with their current status (running/stopped). | The fastest way to grab a VM's ID number or check its power state without opening the web GUI. |
| `qm start <vmid>` | powers on the specified Virtual Machine by its numerical ID. | Allows you to spin up a server remotely via SSH. |
| `qm shutdown <vmid>` | Sends a graceful ACPI shutdown signal to the guest operating system. | Safely stops database and system processes. *Note: Requires the QEMU Guest Agent to be installed and enabled inside the guest OS.* |
| `qm stop <vmid>` | Performs an immediate, hard power-off on the specified VM. | Use this as a last resort if a VM hangs or freezes and refuses to respond to a normal shutdown command. |
| `qm config <vmid>` | Displays the complete hardware configuration file for the specified VM. | Shows allocation details like assigned CPU cores, RAM limits, network bridges, and virtual disk paths. |
| `qm set <vmid> --memory 4096` | Updates the VM's RAM allocation (defined in Megabytes) on the fly. | Allows you to scale a VM's memory up or down. *Note: Changes may require a VM reboot depending on your settings.* |
| `qm terminal <vmid>` | Attaches your current SSH session directly to the VM's serial console. | Grants command-line access to a VM even if its network interfaces are broken or misconfigured. |

---

#### 📦 LXC Container Management (`pct`)

| Command | What It Does | Why it Matters |
| :--- | :--- | :--- |
| `pct list` | Lists all Linux Containers (LXCs) running on the Proxmox host. | Quick inventory check for lightweight containerized services. |
| `pct start <ctid>` | Powers on the specified LXC container by its ID. | Instantly spins up a containerized service. |
| `pct enter <ctid>` | Opens an interactive root shell terminal directly inside the running container. | Unlike Docker, this drops you straight into the container filesystem instantly without needing to specify interactive flags (`-it`) or shell paths (`sh`/`bash`). |
| `pct config <ctid>` | Dumps the hardware and resource allocation profiles for the container. | Shows you which host directories are mounted, network bridge profiles, and resource limits. |
| `pct set <ctid> --memory 512` | Adjusts the memory limit of the container immediately. | Allows you to adjust resource limits on the fly without needing to restart the container. |

---

#### 💾 Storage, Backup & Disaster Recovery

| Command | What It Does | Why it Matters |
| :--- | :--- | :--- |
| `pvesm status` | Displays the operational status, total space, and current usage of all configured storage backends. | Essential for checking if local directories, ZFS pools, or remote network shares (NFS/SMB) are online and healthy. |
| `vzdump <vmid> --storage <pool>` | Triggers an immediate, standalone snapshot backup of a specific VM or container. | Creates a compressed `.tar` or `.zst` backup file and deposits it into your chosen storage target. |
| `qmrestore <backup_file> <vmid>` | Restores a Virtual Machine from a specific `vzdump` backup archive file. | Essential tool for bare-metal disaster recovery or cloning an existing VM setup onto a new ID. |
| `pct restore <ctid> <backup_file>` | Restores an LXC Container from a specific `vzdump` backup archive file. | **Crucial Distinction:** You must use `pct restore` for container backups; trying to use `qmrestore` on an LXC archive will result in an error. |

---

#### 🛡️ Node Status & Security

| Command | What It Does | Why it Matters |
| :--- | :--- | :--- |
| `pveversion` | Displays the exact running version of Proxmox VE and its underlying Linux kernel. | Crucial info when verifying documentation compatibility or troubleshooting system bugs on forums. |
| `pve-firewall status` | Checks if the built-in Proxmox cluster and node-level firewall rules are actively running. | Instantly verifies whether your software-defined datacenter security policies are actively filtering traffic. |
