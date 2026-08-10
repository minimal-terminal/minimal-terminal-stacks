# Watchtower

Watchtower is an indispensable tool that automatically monitors your running Docker containers for new image versions. When a new version is detected, Watchtower pulls the latest image, gracefully stops the old container, and restarts it with the new image, ensuring your services are always up-to-date. It can also clean up old images to save valuable disk space.

## Prerequisites

- Docker installed and running on your host system.
- Watchtower requires access to the Docker socket (`/var/run/docker.sock`) to interact with the Docker daemon and manage containers.

## Docker Compose Script

```yaml
version: '3.8'
services:
  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    # Configure Watchtower's update schedule.
    # Uncomment and configure one of the following scheduling options:

    # Option 1: Poll interval (e.g., check every 5 minutes = 300 seconds)
    # environment:
    #   - WATCHTOWER_POLL_INTERVAL=300

    # Option 2: Cron schedule (e.g., daily at 4 AM - recommended for most setups)
    environment:
      - WATCHTOWER_SCHEDULE=0 0 4 * * *

    # Command arguments for Watchtower:
    # --cleanup: Remove old images after updating (highly recommended to save disk space)
    # --include-stopped: Also update containers that are currently stopped
    # --revive-stopped: Start stopped containers after updating them
    # --debug: Enable debug logging for troubleshooting
    # --label-enable: Only monitor containers with the 'com.centurylinklabs.watchtower.enable: "true"' label
    command: --cleanup
    restart: unless-stopped

    # Optional: If you want Watchtower to monitor only specific containers, you would add the
    # 'com.centurylinklabs.watchtower.enable: "true"' label to those containers in their own
    # 'labels' section (within their 'deploy' block if applicable, or directly under the service).
    # For Watchtower itself to respect this, you would need to add '--label-enable' to its command.
    # Example for another service's configuration:
    #   my_other_service:
    #     image: my/image
    #     labels:
    #       - com.centurylinklabs.watchtower.enable: "true"
    #     ...
```