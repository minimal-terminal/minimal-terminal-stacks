# Uptime Kuma

Uptime Kuma is an open-source, self-hosted monitoring tool that provides an elegant and user-friendly interface to monitor the uptime of your services. It supports various monitoring types (HTTP(s), TCP, Ping, DNS, Push, etc.) and offers multiple notification channels, as well as beautiful status pages.

## Prerequisites

Before deploying Uptime Kuma, ensure you have Docker and Docker Compose installed on your system. You will need a persistent volume for Uptime Kuma's data to ensure your monitoring configurations and history are retained across container restarts.

*   **Volume:** A directory on your host machine to store `/app/data` for Uptime Kuma.
*   **Port:** An available port on your host, typically `3001`, to expose Uptime Kuma's web interface.

## Docker Compose Script

```yaml
version: '3.8'

services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    ports:
      - "3001:3001" # Host_Port:Container_Port - Access via http://your-ip:3001
    volumes:
      - ./uptime-kuma-data:/app/data # Persistent storage for Uptime Kuma's database and configurations
    restart: unless-stopped # Ensure the container restarts automatically unless explicitly stopped
```