# Flame

Flame is a self-hosted startpage that allows you to easily manage and access your favorite applications and links from a single, beautiful dashboard. It's designed for simplicity and speed, making it an ideal central point for your home lab or server.

## Prerequisites

Standard Docker installation, a dedicated volume for persistent data, and an available port on your host system (e.g., `1025`). Ensure your chosen port does not conflict with existing services.

## Docker Compose Script

```yaml
version: '3.8'
services:
  flame:
    image: pawelmalak/flame:latest
    container_name: flame
    volumes:
      - ./data:/app/data # Persistent data for configurations and settings
    ports:
      - "1025:5000" # Host_Port:Container_Port - Access Flame via http://your-server-ip:1025
    environment:
      # Optional: Set PUID and PGID for proper file permissions
      # - PUID=1000
      # - PGID=1000
      - TZ=Etc/UTC # Set your timezone, e.g., America/New_York
    restart: unless-stopped
```