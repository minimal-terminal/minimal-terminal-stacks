# Navidrome

Navidrome is an open source music server and streamer. It gives you freedom to listen to your music collection from anywhere, by using any compatible client. It's like your personal Spotify, but with your own music, fully self-hosted.

## Prerequisites

Before deploying Navidrome with Docker Compose, ensure you have:

*   A Docker-compatible environment (e.g., Linux server, Synology NAS with Docker).
*   A dedicated host path for Navidrome's configuration and database (e.g., `/path/to/your/navidrome/data`).
*   A dedicated host path for your music files (e.g., `/path/to/your/music/library`). This volume should be mounted as read-only (`:ro`) to prevent accidental modifications by the container.
*   Port `4533` (Navidrome's default) available on your host.

## Docker Compose Script

```yaml
version: '3.8'

services:
  navidrome:
    image: deluan/navidrome:latest
    container_name: navidrome
    ports:
      - "4533:4533"
    environment:
      # Scan schedule: '1h' for hourly, '24h' for daily, or '0' to disable periodic scans
      - ND_SCANSCHEDULE=1h
      # Log level: 'info', 'warn', 'error', 'debug'
      - ND_LOGLEVEL=info
      # Session timeout: '24h' or '7d'
      - ND_SESSIONTIMEOUT=24h
      # Folder where Navidrome stores its database and configuration
      - ND_DATAFOLDER=/data
      # Folder where your music is located inside the container
      - ND_MUSICFOLDER=/music
      # Optional: Set a base URL if you're using a reverse proxy (e.g., /navidrome)
      # - ND_BASEURL=/
      # Optional: Set a custom timezone (e.g., Europe/London, America/New_York)
      # - TZ=Etc/UTC
    volumes:
      # Host path for Navidrome's data (database, cache, etc.)
      - /path/to/your/navidrome/data:/data
      # Host path for your music library (read-only)
      - /path/to/your/music/library:/music:ro
    restart: unless-stopped
```