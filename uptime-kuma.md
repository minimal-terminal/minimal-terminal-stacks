# Uptime Kuma

Uptime Kuma is a fancy self-hosted monitoring tool. Monitor your services via HTTP(s), TCP, Ping, DNS, and more. Get notifications via Telegram, Discord, Slack, Email, and many other services. Beautiful status page included.

## Prerequisites

Before deploying Uptime Kuma, ensure you have:

*   A Docker-enabled host (Linux, macOS, Windows via WSL2).
*   A persistent volume for data storage (e.g., a dedicated folder on your NAS or server).
*   An available network port (e.g., `3001`) for its web interface.

## Docker Compose Script

Create a `docker-compose.yml` file in your desired deployment directory and populate it with the following content. Adjust the volume path as necessary to store your data persistently.

```yaml
version: '3.8'

services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - ./uptime-kuma-data:/app/data
    ports:
      - "3001:3001"
    restart: unless-stopped
```