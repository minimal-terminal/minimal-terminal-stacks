# Uptime Kuma

Uptime Kuma is a fancy self-hosted monitoring tool similar to "Uptime Robot" but completely open-source. It allows you to monitor HTTP(s) / TCP / Ping / DNS / Push / Steam Game Server, and more. Get notifications via Telegram, Discord, Gotify, Slack, Email (SMTP), and many other services.

## Prerequisites

Before deploying Uptime Kuma, ensure you have:
*   Docker and Docker Compose installed on your host system.
*   A dedicated directory for Uptime Kuma's persistent data (e.g., `./uptime-kuma-data`).
*   An available port on your host system (defaulting to `3001`) that you wish to expose Uptime Kuma on.

## Docker Compose Script

To deploy Uptime Kuma, save the following content as `docker-compose.yml` in your chosen directory and run `docker compose up -d`.

```yaml
version: '3.8'

services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - ./uptime-kuma-data:/app/data
    ports:
      - 3001:3001 # Host_Port:Container_Port
    restart: unless-stopped
```