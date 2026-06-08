| Command | Purpose | Why it Matters |
| :--- | :--- | :--- |
| `sudo` | Super User Do | Grants admin/root privileges for system changes. |
| `apt update` | Refresh Repo | Syncs local package list with remote repositories. |
| `docker run` | Start Container | Deploys services with flags like `-d` (background) and `-p` (ports). |
| `timedatectl` | Time Mgmt | Ensures logs and cron jobs sync with the correct timezone. |
| `ansible ... -m cron` | Automation | Creates scheduled tasks for maintenance windows. |
| `ls -lah` | List with permissions, human-readable sizes, hidden files |
| `cd -` | Jump back to previous directory |
| `rsync -av src/ dest/` | Sync directories — better than cp for backups |
| `find / -name "*.conf" 2>/dev/null` | Find files by name, suppress permission errors |
| `chmod +x script.sh` | Make a script executable |
| `grep -r "pattern" /path` | Recursive string search across files |
| `grep -E ':80\|:443'` | Extended regex — match multiple patterns |
| `cat file \| jq '.'` | Pretty-print JSON output |
| `awk '{print $2}'` | Extract the second column from output |
| `tail -f /var/log/syslog` | Follow a log file live |
| `$(command)` | Command substitution — use output as a value |
| `if [[ "$VAR" == "val" ]]; then` | Conditional — use `[[ ]]` for strings, not `[ ]` |
| `while true; do ... sleep 2; done` | Infinite loop with pause — used in the ntfy bridge |
| `curl -s --max-time 90 "$URL"` | Silent curl with timeout — essential for long-polling |
| `2>/dev/null` | Discard stderr — keeps script output clean |
| `crontab -e` | Edit current user's cron jobs |
| `0 2 * * * /path/script.sh` | Run at 2:00 AM every day |
| `*/5 * * * * /path/script.sh` | Run every 5 minutes |


# Open the terminal text editor
nano filename.txt

# Trace the route and IP hops to a target (no DNS resolution)
tracert -d cisco.com        # Windows
traceroute -n cisco.com     # Linux
