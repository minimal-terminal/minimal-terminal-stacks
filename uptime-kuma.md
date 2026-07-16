# Uptime Kuma

Uptime Kuma is a free and open-source self-hosted monitoring tool like "Uptime Robot" but with a beautiful dashboard, extensive notification options, and a user-friendly interface. It allows you to monitor HTTP(s), TCP, Ping, DNS, and more, providing instant alerts and elegant status pages.

## Prerequisites

*   A Docker-compatible environment (e.g., Linux server, Synology NAS with Docker).
*   Ensure port `3001` (or your chosen host port) is available on your server.
*   A dedicated directory for Uptime Kuma's data persistence (e.g., `./uptime-kuma-data`).

## Docker Compose Script

```yaml
version: '3.8'

services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - ./uptime-kuma-data:/app/data
    ports:
      - "3001:3001" # Host_Port:Container_Port - Access Uptime Kuma via http://your-server-ip:3001
    restart: unless-stopped
```