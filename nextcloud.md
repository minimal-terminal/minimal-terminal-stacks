# Nextcloud

Nextcloud is a suite of client-server software for creating and using file hosting services. It is functionally similar to Dropbox, Google Drive, OneDrive, or ownCloud. Nextcloud is free and open-source, which means anyone can install and operate it on a private server with full control over their data.

## Prerequisites

Before deploying Nextcloud, ensure you have Docker and Docker Compose installed. You will need to create Docker volumes for persistent data storage and map appropriate ports. This setup uses a PostgreSQL database for robust performance and a Redis cache for improved responsiveness.

*   **Docker Volumes**:
    *   `nextcloud_data`: For Nextcloud application data, files, and configurations.
    *   `nextcloud_db`: For PostgreSQL database files.
    *   `nextcloud_redis`: For Redis cache data.
*   **Ports**:
    *   `80:80`: Standard HTTP port for Nextcloud web access (can be mapped to a different external port if 80 is in use, or routed through a reverse proxy).

## Docker Compose Script

```yaml
version: '3.8'

volumes:
  nextcloud_data:
    driver: local
  nextcloud_db:
    driver: local
  nextcloud_redis:
    driver: local

services:
  db:
    image: postgres:15-alpine
    restart: unless-stopped
    environment:
      POSTGRES_DB: nextcloud
      POSTGRES_USER: nextclouduser
      POSTGRES_PASSWORD: your_strong_db_password # <--- CHANGE THIS
    volumes:
      - nextcloud_db:/var/lib/postgresql/data
    networks:
      - nextcloud_network

  redis:
    image: redis:7-alpine
    restart: unless-stopped
    volumes:
      - nextcloud_redis:/data
    networks:
      - nextcloud_network

  app:
    image: nextcloud:latest # Uses the official image with Apache pre-configured
    restart: unless-stopped
    ports:
      - "80:80" # Map host port 80 to container port 80. Adjust if 80 is in use.
      # If you are using a reverse proxy like Nginx Proxy Manager, you might only
      # need to expose port 80 internally and not map it to the host, or map
      # it to a high host port (e.g., 8080:80) and configure your proxy.
    links:
      - db
      - redis
    volumes:
      - nextcloud_data:/var/www/html
    environment:
      NEXTCLOUD_TRUSTED_DOMAINS: localhost,your.domain.com # <--- CHANGE THIS to your actual domain
      NEXTCLOUD_OVERWRITEPROTOCOL: https # Use https if behind a reverse proxy
      POSTGRES_HOST: db
      POSTGRES_DB: nextcloud
      POSTGRES_USER: nextclouduser
      POSTGRES_PASSWORD: your_strong_db_password # <--- CHANGE THIS to match db service
      REDIS_HOST: redis
      REDIS_HOST_PORT: 6379
      # For a production setup, consider defining memory limits and other optimizations.
    networks:
      - nextcloud_network

networks:
  nextcloud_network:
    driver: bridge
```