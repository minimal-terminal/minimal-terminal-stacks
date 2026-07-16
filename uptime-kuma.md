# Uptime Kuma

Uptime Kuma is a free and open-source monitoring tool. It allows you to monitor services over HTTP(s), TCP, Ping, DNS, and more, providing beautiful status pages and various notification options.

## Prerequisites

*   Docker and Docker Compose installed.
*   A dedicated directory for Uptime Kuma data, e.g., `/path/to/your/uptime-kuma/data`.
*   An available port on your host, typically `3001`, to expose Uptime Kuma's web interface.

## Docker Compose Script

```yaml
version: '3.8'

services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - /path/to/your/uptime-kuma/data:/app/data
    ports:
      - "3001:3001"
    restart: unless-stopped
```