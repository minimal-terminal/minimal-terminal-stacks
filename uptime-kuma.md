# Uptime Kuma

Uptime Kuma is a fancy, self-hosted monitoring tool that helps you keep an eye on the uptime of your websites, services, and APIs. It provides a beautiful and intuitive dashboard, supporting various monitoring protocols and offering instant notifications for downtime.

## Prerequisites

Ensure Docker and Docker Compose are installed on your system. A dedicated directory for Uptime Kuma's persistent data (e.g., `uptime-kuma-data`) is recommended to prevent data loss upon container recreation.

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
      - "3001:3001" # HostPort:ContainerPort - Change 3001 to your desired host port
    restart: unless-stopped
    # If you need to specify a network, uncomment and configure below
    # networks:
    #   - my-docker-network

# If you uncommented the networks section above, define your network here
# networks:
#   my-docker-network:
#     external: true
#       name: my-docker-network
```