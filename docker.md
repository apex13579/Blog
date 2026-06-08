### 🐳 Docker Container Management

#### 🔄 Container Lifecycle

| Command | What It Does | Why it Matters |
| :--- | :--- | :--- |
| `docker ps` | Lists currently running containers. | Shows you what services are active right now and how long they have been up. |
| `docker ps -a` | Lists all containers (both running and stopped). | Essential for finding broken containers that exited or failed to start. |
| `docker stop $(docker ps -aq)` | Gracefully stops every single running container simultaneously. | The fastest way to clear the deck before performing host updates or system reboots. |
| `docker start <name>` | Boots up a stopped container while preserving its previous configuration. | Brings a service back online without needing to re-type a long `docker run` command. |
| `docker restart <name>` | Stops and immediately restarts a container. | Forces configuration file reloads and clears stuck application processes. |
| `docker rm -f <name>` | Force-deletes a container, even if it is currently running. | Instantly destroys a container so you can rebuild it from scratch without stopping it first. |

---

#### 🚀 Running Containers

| Command | What It Does | Why it Matters |
| :--- | :--- | :--- |
| `docker run -d <image>` | Launches a container detached in the background. | Frees up your terminal window so the service runs silently as a background daemon. |
| `docker run -p 8080:80 <image>` | Maps ports using **HOST : CONTAINER** format. | Tells Docker: "When traffic hits my server's port 8080, route it to the container's port 80." |
| `docker run -v vol:/path <image>` | Mounts a persistent volume using **VOLUME : CONTAINER_PATH**. | Prevents data loss. Ensures databases or app data survive when the container is deleted or updated. |
| `docker run -e VAR=value <image>` | Injects an environment variable into the container. | Used to securely pass runtime configurations like timezones, database passwords, or API keys. |
| `docker run --restart=unless-stopped` | Sets an automatic restart policy for the container. | Ensures your services spin back up automatically if the host server reboots or crashes. |

---

#### 🛠️ Troubleshooting & Diagnostics

| Command | What It Does | Why it Matters |
| :--- | :--- | :--- |
| `docker logs -f <name>` | Streams a container's internal log output in real-time. | The ultimate debugging command for watching application errors or connection attempts live. |
| `docker inspect <name>` | Dumps the full, low-level JSON metadata of a container. | Used to find internal IP addresses, exact volume paths, and environment variables. |
| `docker exec -it <name> sh` | Drops you directly into an interactive shell inside the container. | Allows you to navigate the container's internal filesystem. *Note: Use `sh` instead of `bash` for minimal/Alpine images.* |
| `docker stats` | Displays a live, streaming resource usage dashboard for all containers. | Quickly identifies memory leaks or rogue containers that are hogging CPU and RAM. |
| `docker system prune -a` | **Warning:** Nukes all stopped containers, unused networks, and cached images. | A nuclear option to instantly reclaim gigabytes of disk space when your drive fills up. |

---

#### 💾 Volumes & Networking

| Command | What It Does | Why it Matters |
| :--- | :--- | :--- |
| `docker volume ls` | Lists every named storage volume managed by Docker. | Helps track down abandoned volumes that are taking up storage space after containers are deleted. |
| `docker volume create <name>` | Manually provisions an isolated storage directory. | Allows you to pre-stage persistent storage spaces before attaching them to new containers. |
| `docker network ls` | Lists all isolated virtual networks built on the host. | Lets you see what communication lanes exist (e.g., bridge, host, macvlan) for your containers. |
| `docker network inspect bridge` | Inspects the default bridge network to show its attached containers. | Shows you exactly which containers can talk to one another and lists their internal subnet IPs. |
