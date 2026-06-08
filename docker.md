### Docker

#### Container Lifecycle

| Command | What It Does |
|---|---|
| `docker ps -a` | List all containers — running and stopped |
| `docker stop $(docker ps -aq)` | Stop every running container at once |
| `docker start <name>` | Start a stopped container by name |
| `docker restart <name>` | Stop and start — reloads config |
| `docker rm -f <name>` | Force-delete even a running container |

#### Running Containers

| Command | What It Does |
|---|---|
| `docker run -d` | Run detached (background) |
| `docker run -p 8080:80` | Map host port 8080 → container port 80 |
| `docker run -v vol:/path` | Mount a named volume into the container |
| `docker run -e VAR=value` | Pass an environment variable |
| `docker run --restart=unless-stopped` | Auto-restart unless manually stopped |

#### Debugging

| Command | What It Does |
|---|---|
| `docker logs -f <name>` | Tail live logs from a container |
| `docker inspect <name>` | Full metadata — ports, mounts, network config |
| `docker exec -it <name> bash` | Drop into a running container's shell |
| `docker stats` | Live CPU and RAM usage per container |
| `docker system prune -a` | Nuke all stopped containers, unused images and networks |

#### Volumes & Networks

| Command | What It Does |
|---|---|
| `docker volume ls` | List all named volumes |
| `docker volume create <name>` | Create a named volume |
| `docker network ls` | List all Docker networks |
| `docker network inspect bridge` | Inspect default bridge + attached containers |
