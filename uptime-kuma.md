# Uptime Kuma

Uptime Kuma is a fancy self-hosted monitoring tool like 'Uptime Robot' but better. It monitors HTTP(s), TCP, Ping, DNS, Push, Steam Game Server, Docker Containers, and more. Get notifications via Telegram, Discord, Email, and many other services. Beautiful, user-friendly UI.

## Prerequisites

*   Docker and Docker Compose installed.
*   A dedicated directory for Uptime Kuma's persistent data (e.g., `~/docker/uptime-kuma/data`).
*   An available port (e.g., `3001`) on your host machine.

## Docker Compose Script

```yaml
version: '3.3'

services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - ./data:/app/data
    ports:
      - "3001:3001"
    restart: unless-stopped
    networks:
      - kuma-network

networks:
  kuma-network:
    driver: bridge
```