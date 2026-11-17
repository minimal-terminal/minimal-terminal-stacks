---
# Prowlarr - Unified Indexer and Client Management
#
# Description:
# Prowlarr is an indexer manager and proxy for your *arr stack. It integrates
# directly with Sonarr, Radarr, Lidarr, and Readarr, providing a centralized
# platform for managing your Usenet indexers and BitTorrent trackers. This
# streamlines configuration and ensures all your media automation tools
# have access to the same robust set of sources.
#
# Prerequisites:
# - Docker and Docker Compose installed.
# - A dedicated directory for Prowlarr's configuration data (e.g., /srv/docker/prowlarr/config).
# - Ensure PUID and PGID match your unprivileged user for proper file permissions.
#
version: "3.8"
services:
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment:
      - PUID=1000 # User ID for permissions (e.g., `id -u youruser`)
      - PGID=1000 # Group ID for permissions (e.g., `id -g youruser`)
      - TZ=Etc/UTC # Set your timezone, e.g., America/New_York
    volumes:
      - /path/to/your/prowlarr/config:/config # Persistent configuration data
    ports:
      - "9696:9696" # Web UI access (Host:Container)
    restart: unless-stopped
