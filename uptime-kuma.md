# Uptime Kuma

Uptime Kuma is a free and open-source self-hosted monitoring tool like "Uptime Robot" but with more features. Monitor your services via HTTP(s), TCP, Ping, and more, and get notifications via various channels. It's perfect for ensuring your self-hosted applications and services are always online.

## Prerequisites

Before deploying Uptime Kuma, ensure you have:

*   Docker and Docker Compose installed on your host system.
*   An available port (e.g., `3001`) on your host machine for Uptime Kuma's web interface.
*   A dedicated Docker volume for persistent data storage, ensuring your monitoring configurations and history are retained across container restarts.

## Docker Compose Script

```yaml
version: '3.8'

services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - uptime-kuma-data:/app/data
    ports:
      - "3001:3001" # Host_Port:Container_Port
    restart: unless-stopped
    # Optional: Set environment variables if needed, e.g., for proxy setup
    # environment:
    #   - PUID=1000
    #   - PGID=1000

volumes:
  uptime-kuma-data:
    # Optional: Specify a driver and driver_opts if you need advanced volume configurations
    # driver: local
    # driver_opts:
    #   type: "none"
    #   o: "bind"
    #   device: "/path/to/your/host/data/for/uptime-kuma"
```