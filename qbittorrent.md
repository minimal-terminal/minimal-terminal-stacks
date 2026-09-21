# qBittorrent

qBittorrent is a free and open-source BitTorrent client that provides a powerful Web UI, allowing users to remotely manage downloads and uploads from their self-hosted environment. It's an excellent tool for streamlining media acquisition for your homelab or NAS.

## Prerequisites

Before deploying qBittorrent with Docker Compose, ensure you have:

*   Docker and Docker Compose installed on your system.
*   A dedicated directory for qBittorrent's configuration and downloaded files.
*   The `PUID` (User ID) and `PGID` (Group ID) of the user that Docker will run as. You can find these by running `id <username>` in your terminal (e.g., `id youruser`). Replace `1000` with your actual PUID/PGID.
*   Your local timezone for the `TZ` environment variable (e.g., `America/New_York`, `Etc/UTC`).

## Docker Compose Script

```yaml
version: "3.8"
services:
  qbittorrent:
    image: linuxserver/qbittorrent:latest
    container_name: qbittorrent
    environment:
      - PUID=1000 # Replace with your user's PUID
      - PGID=1000 # Replace with your user's PGID
      - TZ=Etc/UTC # Replace with your timezone, e.g., America/New_York
      - WEBUI_PORT=8080
    volumes:
      - ./config:/config
      - ./downloads:/downloads
    ports:
      - 8080:8080 # Web UI
      - 6881:6881 # Torrent TCP port
      - 6881:6881/udp # Torrent UDP port
    restart: unless-stopped
```