# Sonarr

Sonarr is an intelligent PVR (Personal Video Recorder) for Usenet and BitTorrent users. It can monitor multiple RSS feeds for new episodes of your favorite TV shows and will automatically grab, sort, and rename them. It integrates seamlessly with your download clients and media servers, ensuring your TV library is always up-to-date and perfectly organized.

## Prerequisites

Before deploying Sonarr, ensure you have:

*   **Docker and Docker Compose** installed on your system (e.g., Synology NAS, Ubuntu server).
*   **Volumes Mapped**: Dedicated paths for Sonarr's configuration, your TV show library, and a temporary downloads folder (where your download client places completed files).
*   **User and Group IDs (PUID/PGID)**: Determine your user's PUID and PGID by running `id -u` and `id -g` respectively, to ensure correct file permissions within the container.
*   **Timezone**: Set your local timezone for accurate scheduling.

## Docker Compose Script

```yaml
version: "3.8"
services:
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=1000 # IMPORTANT: Replace with your user ID (e.g., 1000)
      - PGID=1000 # IMPORTANT: Replace with your group ID (e.g., 1000)
      - TZ=Etc/UTC # IMPORTANT: Replace with your timezone (e.g., America/New_York)
    volumes:
      - ./sonarr/config:/config # Path to store Sonarr's configuration files
      - /path/to/your/tvseries:/tv # IMPORTANT: Path to your TV show library on the host
      - /path/to/your/downloads:/downloads # IMPORTANT: Path where your download client places completed files
    ports:
      - 8989:8989 # Web UI port: Host_Port:Container_Port
    restart: unless-stopped
    # Optionally, you can add a network if you're using a custom Docker network
    # networks:
    #   - your_custom_network

# Optionally, define custom networks if used
# networks:
#   your_custom_network:
#     external: true
```