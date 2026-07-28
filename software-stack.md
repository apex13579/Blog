# **With the upcoming lack of time to give the lab the attention it deserves I will be revamping this plan. I will have forgotten where I left off so I will be starting from scratch.**
## VM machine (Proxmox)
### VM (Falco and Node exporter)
- Active directory (Ubuntu, 2C, 2R, 50S)
- Home assistant (Home assistant OS, 2C, 2R, 32S)
- Honeypot (Alpine, 1C, 1R, 15S)
- Suricata (Alpine, 2C, 4R, 40S)
### LXC
- Dockhand (1C, .5R, 15S)
- MATIJAZEZELJ/SIB (4C, 6R, 80S)
- pangolin newt (1C, .5R, 4S)
- Authelia (1C, .5R, 4S)
- Unifi controller (2C, 1R, 8S)
- Vault warden (1C, .5R, 4S)
- Gitea (2C, 1R, 15S)
- Ansible & Terraform (1C, .5R, 4S)
- Prowler & IT tools (1C, 1R, 8S)
- Searxng (1C, 1R, 8S)
- Excalidraw (1C, 1R, 4S)
- Zigbee2MQTT & Mosquitto (1C, .5R, 4S)
- NTFY & Apprise (1C, .5R, 4S)
- Flame (1C, .25R, 2S)
- Grafana & Prometheus (2C, 2R, 15S)
- Falco sidekick (1C, .5R, 4S)
- Nextcloud (2C, 1R, 20S)
- Pi-hole (1c, .5R, 4S)
- Linkding (1C, .5R, 4S)
- Rsync (1C, .5R, 2S)
- PBS (2C, 2R, 50S)
- PhpIPAM (1C, 1R, 8S)
- Cups (1C, .5R, 20S)
- Traefik (1C, .5R, 4S)
## NAS (unraid with built in prometheus exporter)
- ARR Stack
  - Radarr
  - Bazarr
  - Jellyseerr
  - Lidarr
  - Readarr
  - Sonarr
  - Prowlarr
- LibrePhotos
- Web dav (as gate for joplin sync)
- Qbittorrent
- NZB client
## HP Elitedesk 800 G3 (RTX A2000 equipped)
- Ollama & opem web ui
- nvidia-smi-exporter
- node exporter
## Dell Optiplex 5040 (Ububtu A310)
- Jellyfin (library on nas)
- Tailscale
- Node exporter
- intel gpu top
## cisco 5512-x (opn-sense with build in telegraf exporter)
- crowdsec
- geoip
## oracle cloud instances
- pangolin 
