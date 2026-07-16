# Uptime Kuma

Uptime Kuma is a free, open-source, and self-hosted monitoring tool like "Uptime Robot" but with a modern UI, push/pull/proxy support, and various notification methods. It allows you to monitor your services and receive notifications when they go down, ensuring your digital infrastructure remains robust and reliable.

## Prerequisites

To deploy Uptime Kuma, you'll need:

*   A Docker-compatible environment (e.g., a Linux server, Synology NAS, or any system with Docker Engine).
*   An available port on your host machine (e.g., 3001) that won't conflict with other services.
*   A dedicated Docker volume for persistent data storage, ensuring your monitoring configurations and history are preserved across container restarts.

## Docker Compose Script

Below is a production-ready `docker-compose.yml` script for deploying Uptime Kuma. Ensure you have Docker and Docker Compose installed on your system.

```yaml
version: '3.8'

services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    ports:
      - "3001:3001" # Host_Port:Container_Port
    volumes:
      - uptime-kuma-data:/app/data
    restart: unless-stopped

volumes:
  uptime-kuma-data:
    driver: local
```