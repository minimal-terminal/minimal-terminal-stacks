# Homepage

Homepage is a modern, fully static, super fast, secure, and highly customizable dashboard for your self-hosted applications. It allows you to organize all your services, applications, and links in one central place, providing live status widgets, search functionality, and a clean interface.

## Prerequisites

Ensure you have Docker and Docker Compose installed on your system. A dedicated volume for Homepage configuration is recommended for persistent data. Homepage typically runs on port 3000, which can be exposed directly or proxied through a reverse proxy (e.g., Nginx Proxy Manager).

## Docker Compose Script

```yaml
version: "3.8"
services:
  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    ports:
      - "3000:3000" # You can change the host port if 3000 is in use
    volumes:
      - ./homepage/config:/app/config # Path to your config directory on the host
      - /var/run/docker.sock:/var/run/docker.sock:ro # Optional: For Docker integration to show container status
    environment:
      PUID: "1000" # Optional: Your User ID, adjust as needed
      PGID: "1000" # Optional: Your Group ID, adjust as needed
      TZ: "Etc/UTC" # Optional: Your Timezone, e.g., "America/New_York"
    restart: unless-stopped
```