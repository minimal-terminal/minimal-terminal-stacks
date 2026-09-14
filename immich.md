# Immich

Immich is a high-performance, self-hosted photo and video backup solution that provides a powerful alternative to cloud-based services like Google Photos. It offers automatic mobile backups, advanced AI-powered search (facial recognition, object detection), multi-user support, and shared albums, giving you full control over your precious memories on your own hardware.

## Prerequisites

Before deploying Immich, ensure you have:

*   Docker and Docker Compose installed on your host system (e.g., Linux server, NAS).
*   Sufficient storage available for your photos and videos. Immich will store these in a dedicated Docker volume.
*   A `.env` file in the same directory as your `docker-compose.yml` with the following variables:

    ```ini
    # Database Configuration
    DB_HOSTNAME=immich-postgres
    DB_USERNAME=immich
    DB_PASSWORD=your_secure_database_password
    DB_DATABASE=immich
    DB_PORT=5432

    # Redis Configuration
    REDIS_HOSTNAME=immich-redis
    REDIS_PORT=6379

    # Immich Server & Microservices Configuration
    IMMICH_SERVER_URL=http://immich-server:3001
    UPLOAD_LOCATION=/usr/src/app/upload

    # Immich Proxy Configuration (optional, but recommended for clean setup)
    # No specific variables usually needed unless custom proxy configuration is used
    ```

    **Important**: Replace `your_secure_database_password` with a strong, unique password.

## Docker Compose Script

```yaml
version: "3.8"

services:
  immich-postgres:
    container_name: immich_postgres
    image: postgres:15
    env_file:
      - .env
    environment:
      POSTGRES_DB: ${DB_DATABASE}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      # PG_LISTEN_ADDRESSES: '*' # Uncomment if you need external access to Postgres (not typically needed for Immich)
    volumes:
      - immich_pgdata:/var/lib/postgresql/data
    restart: unless-stopped
    networks:
      - immich_network

  immich-redis:
    container_name: immich_redis
    image: redis:6.2-alpine
    env_file:
      - .env
    volumes:
      - immich_redisdata:/data
    restart: unless-stopped
    networks:
      - immich_network

  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:release
    command: ["start-server.sh"]
    env_file:
      - .env
    volumes:
      - immich_library:/usr/src/app/upload # This volume stores your actual photos and videos
    depends_on:
      - immich-postgres
      - immich-redis
    restart: unless-stopped
    networks:
      - immich_network

  immich-microservices:
    container_name: immich_microservices
    image: ghcr.io/immich-app/immich-microservices:release
    command: ["start-microservices.sh"]
    env_file:
      - .env
    volumes:
      - immich_library:/usr/src/app/upload
    depends_on:
      - immich-postgres
      - immich-redis
    restart: unless-stopped
    networks:
      - immich_network

  immich-web:
    container_name: immich_web
    image: ghcr.io/immich-app/immich-web:release
    env_file:
      - .env
    depends_on:
      - immich-server
    restart: unless-stopped
    networks:
      - immich_network

  immich-proxy:
    container_name: immich_proxy
    image: ghcr.io/immich-app/immich-proxy:release
    env_file:
      - .env
    ports:
      - "2283:80" # Exposes the Immich web UI on host port 2283
    depends_on:
      - immich-server
      - immich-web
    restart: unless-stopped
    networks:
      - immich_network

networks:
  immich_network:
    driver: bridge

volumes:
  immich_pgdata:
  immich_redisdata:
  immich_library:
```