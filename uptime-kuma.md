# Uptime Kuma

Uptime Kuma is a fancy self-hosted monitoring tool like "Uptime Robot". It monitors HTTP(s), TCP, Ping, DNS, Push, Steam Game Server, and more. Get notifications via Telegram, Discord, Slack, Email (SMTP), Pushbullet, Pushover, Microsoft Teams, Webhook, and many more.

## Prerequisites

*   Docker and Docker Compose installed.
*   A dedicated directory for Uptime Kuma's persistent data (e.g., `~/docker/uptime-kuma/data`).
*   An available port on your host (e.g., `3001`) for the Uptime Kuma web interface.

## Docker Compose Script

```yaml
version: '3.8'

services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    ports:
      - "3001:3001" # Host_Port:Container_Port
    volumes:
      - ./data:/app/data # Persistent data storage
    restart: unless-stopped
    environment:
      - PUID=1000 # Optional: User ID for permissions
      - PGID=1000 # Optional: Group ID for permissions
      - TZ=Etc/UTC # Optional: Timezone
```