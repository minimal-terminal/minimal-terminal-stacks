# Docker Container Health Restarter (Bash)

Your self-hosted infrastructure deserves to be resilient. This simple yet powerful Bash script continuously monitors the health of your Docker containers and automatically restarts any that are flagged as `unhealthy`. This proactive approach minimizes downtime and ensures your critical services remain online without constant manual intervention.

## Prerequisites

*   A Linux environment with `bash` and `docker` installed.
*   Docker containers configured with `HEALTHCHECK` instructions in their `Dockerfile` or `docker-compose.yml` to report their health status.

## The Script: `docker_auto_heal.sh`

```bash
#!/bin/bash
# docker_auto_heal.sh - Restarts unhealthy Docker containers based on Docker health checks

echo "🔍 Initiating Docker container health scan..."

# Iterate through all running Docker containers
docker ps --format "{{.ID}}\t{{.Names}}" | while IFS=$'\t' read -r ID NAME; do
    # Retrieve the health status of the container
    # 2>/dev/null suppresses errors for containers without health checks
    HEALTH_STATUS=$(docker inspect --format "{{.State.Health.Status}}" "$ID" 2>/dev/null)

    case "$HEALTH_STATUS" in
        "unhealthy")
            echo "🚨 Unhealthy: $NAME ($ID) - Restarting container..."
            docker restart "$ID" >/dev/null # Redirect output to /dev/null for cleaner logs
            ;;
        "healthy")
            echo "✅ Healthy: $NAME"
            ;;
        "starting")
            echo "⏳ Starting: $NAME"
            ;;
        *) 
            # Covers containers without a health check defined or other states (e.g., 'none')
            echo "❓ Status: ${HEALTH_STATUS:-"No Health Check"} - $NAME"
            ;;
    esac
done

echo "✨ Docker health scan complete."
```

## Execution Instructions

1.  **Save the Script**:
    Create a new file, for example, `docker_auto_heal.sh`, and paste the code above into it.

    ```bash
    nano docker_auto_heal.sh
    ```

2.  **Make Executable**:
    Grant execute permissions to the script:

    ```bash
    chmod +x docker_auto_heal.sh
    ```

3.  **Run Manually (Test)**:
    Execute the script to see it in action:

    ```bash
    ./docker_auto_heal.sh
    ```

4.  **Automate with Cron (Recommended)**:
    For continuous monitoring, schedule the script to run periodically using cron.
    Edit your crontab:

    ```bash
    crontab -e
    ```

    Add the following line to run the script every 5 minutes (adjust path as needed):

    ```cron
    */5 * * * * /path/to/your/docker_auto_heal.sh >> /var/log/docker_auto_heal.log 2>&1
    ```
    This will run the script every 5 minutes and append its output to `/var/log/docker_auto_heal.log`.

This script is a robust solution to maintain the uptime and stability of your Dockerized services. Ensure your containers are properly configured with health checks for optimal results.
