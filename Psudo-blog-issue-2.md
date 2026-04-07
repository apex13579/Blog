Homelab Overhaul: Building a Fresh Command & Control Stack on Linux Mint (Deep Technical Walkthrough)

I’ve continued my homelab overhaul with a fresh base stack running on Linux Mint. The first VM I built is my Command and Control node, which acts as the central point for deploying, managing, and monitoring the rest of the environment. This VM runs on my Proxmox hypervisor with 4 vCPUs, a 40GB virtual disk, and 4GB of RAM (3814.697 MiB). I enabled the QEMU guest agent and configured the VM for long‑term stability. My design philosophy is simple: keep the node as lightweight as possible so it remains responsive even under load. That means containerizing nearly everything.

The first layer of the stack is Docker. Installing docker.io provides the Docker Engine, the CLI, containerd, and the systemd service unit. After installation, I verify that the Docker daemon is active and enabled so it starts automatically on reboot. Docker is the foundation for everything else in this node, so ensuring the service is healthy is critical.

With Docker running, the next component I deploy is Portainer. Portainer is a lightweight container orchestration and management GUI that sits on top of Docker. It allows me to visually inspect containers, volumes, networks, logs, and resource usage. Before running Portainer, I create a persistent Docker volume called portainer_data. This ensures that Portainer’s configuration survives container updates or recreation. When running Portainer, I bind port 9443 on the host to port 9443 inside the container, assign the container a name, configure it to always restart, and mount both the Docker socket and the persistent data volume. Mounting the Docker socket gives Portainer direct access to the Docker API, which is how it manages the engine. Once the container is running, I can access Portainer through a browser at https://<node-ip>:9443. The first‑time setup requires creating an admin password with a minimum length of 12 characters.

The next service I deploy is Vaultwarden, a lightweight Bitwarden‑compatible password manager written in Rust. Vaultwarden is extremely resource‑efficient, making it ideal for homelab use. I create a persistent volume called vaultwarden_data and then run the Vaultwarden container with a restart policy, a volume mount for persistent data, and a port mapping from host port 8080 to container port 80. Vaultwarden becomes accessible through the browser, and from there I can set up my master password and begin storing credentials. I strongly recommend using a long master passphrase and enabling periodic reminders to rotate passwords every 3–6 months. Good password hygiene is one of the easiest ways to avoid ending up in data breach dumps.

At this point, both Portainer and Vaultwarden should appear under the Images and Containers sections in Portainer. If I ever need to reset the Portainer admin password, I can stop the Portainer container, run the helper-reset-password utility using the portainer_data volume, and then restart the container. The helper will output a temporary password that can be used to log back in.

The next service I install is ntfy.sh, which I use for push notifications related to hardware or software issues. Running ntfy in a container binds port 80 on the host to port 80 inside the container. When ntfy starts, it begins outputting logs directly to the terminal, which is normal behavior. Pressing Ctrl+C stops the foreground process and returns control to the shell. Because many services bind to specific ports, I maintain a port chart that lists each service and the port it uses. This prevents conflicts later when adding more services.

After setting up the core services, I move on to installing Ansible. Ansible will handle automated nightly updates for the node. Before installing Ansible, I update and upgrade the system to ensure all packages are current. I then install software-properties-common, which allows me to add external repositories. I add the official Ansible PPA, update again so the package list includes the new repository, and then install Ansible. Once installed, I create an Ansible playbook called update_system.yml using Nano. The playbook defines a single task: update all packages to their latest version, clean the package cache, and remove unused dependencies. After saving the playbook, I verify the system timezone using timedatectl. If the timezone is incorrect, I set it to the appropriate region (in my case, America/New_York).

With the playbook ready and the timezone correct, I create a cron job using Ansible’s cron module. The cron job is configured to run at 2:00 AM every night, which is a safe maintenance window since no one uses my services at that hour. The cron job executes the Ansible playbook directly. The -K flag is required because the cron module needs privilege escalation to write the cron entry.

On the certification side of things, I’ve now completed 2/9 of the Google Data Analytics course. I’ll continue progressing through it alongside the homelab rebuild.

Bash Cheatsheet (Commands Used in This Post)

sudo
Meaning: “super user do.”
Purpose: Runs a command with administrative/root privileges.
Why it matters: Many system-level operations require elevated permissions.

apt update
Meaning: Refresh the package list from repositories.
Purpose: Lets the system know what updates are available.

apt upgrade
Meaning: Install the newest versions of all installed packages.
Purpose: Keeps the system up to date.

apt install <package>
Meaning: Install a package from the repositories.
Purpose: Used to install software such as docker.io or ansible.

docker volume create <name>
Meaning: Create a persistent storage volume for a container.
Purpose: Ensures data survives container updates or recreation.

docker run
Meaning: Create and start a new container.
Common flags:
-d = run in background
-p = map host port to container port
--name = assign container name
--restart=always = auto-start on reboot
-v = mount a volume into the container

docker stop <container>
Meaning: Stop a running container.

docker start <container>
Meaning: Start a stopped container.

docker systemctl enable --now docker
Meaning: Enable Docker to start at boot and start it immediately.

timedatectl
Meaning: Display the system’s current time and timezone.

sudo timedatectl set-timezone <Region/City>
Meaning: Set the system timezone.

nano <filename>
Meaning: Open a text file in the Nano editor.
Controls:
Ctrl+O = save
Enter = confirm
Ctrl+X = exit

ansible localhost -m cron
Meaning: Use Ansible to create or modify a cron job.
Purpose: Automates scheduled tasks.
