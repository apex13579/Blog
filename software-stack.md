# My Software Stack

This page documents the software and tools I use in my homelab for virtualization, containerization, automation, and services.

## Virtualization & OS
- **Proxmox** - Hypervisor and VM management platform
- **Linux Mint** - Lightweight Linux distribution

## Containerization & Orchestration
- **Docker** - Container runtime
- **Portainer** - Docker management UI and container orchestration

## Services & Applications
- **Vaultwarden** - Bitwarden-compatible password manager
- **ntfy.sh** - Notification service
- **traefik** - Acts as my edge router and reverse proxy, handling SSL termination and routing traffic to my containerized services.
- **Tailscale** - My zero-config Mesh VPN. It allows for secure, encrypted remote access to my lab without exposing ports to the public internet.
- **Gitea** - My self-hosted Git service for version control of my configuration files
- **WebDAV** - Serves as a critical data gateway, specifically used to synchronize my Joplin instances for persistent, cross-platform note-taking and documentation.

## Architecture Notes

My setup focuses on lightweight, containerized applications running on Proxmox VMs. Alpine Linux provides minimal resource overhead, while Docker and Portainer simplify deployments and management.

## Future Additions

- [ ] Additional monitoring tools
- [ ] More containerized services
- [ ] Enhanced automation with Ansible playbooks
- [ ] Documentation of key configurations

Check back for detailed guides on setting up and configuring these tools!
