# Paperless-ngx

Paperless-ngx is an open-source document management system that transforms your physical documents into a searchable, digital archive. It automatically recognizes, tags, and organizes your scanned documents, making them easily accessible and searchable through a web interface.

## Prerequisites

*   Docker and Docker Compose installed.
*   A `.env` file in the same directory as `docker-compose.yml` containing `PAPERLESS_SECRET_KEY=your_strong_secret_key`. Generate this key using `openssl rand -hex 32` (e.g., `openssl rand -hex 32` will output a 64-character hexadecimal string).
*   Persistent volumes for data storage (documents, media, database).
*   Open port 8000 for web access.

## Docker Compose Script

```yaml
version: '3.8'

services:
  webserver:
    image: paperless-ngx/paperless-ngx:latest
    container_name: paperless-ngx_webserver
    ports:
      - "8000:8000"
    env_file:
      - .env
    environment:
      # User and Group IDs for file permissions. Adjust as needed for your host system.
      # PUID and PGID can be found using 'id -u <your_user>' and 'id -g <your_user>'
      PUID: 1000
      PGID: 1000
      TZ: Europe/Berlin # Adjust to your timezone, e.g., America/New_York
      PAPERLESS_REDIS: redis://broker:6379
      PAPERLESS_DBHOST: db
      PAPERLESS_DBNAME: paperless
      PAPERLESS_DBUSER: paperless
      PAPERLESS_DBPASS: paperless
      # PAPERLESS_SECRET_KEY is loaded from the .env file
      PAPERLESS_URL: http://localhost:8000 # Adjust if behind a reverse proxy
    volumes:
      - paperless_data:/usr/src/paperless/data
      - paperless_media:/usr/src/paperless/media
      - paperless_consume:/usr/src/paperless/consume
      - paperless_export:/usr/src/paperless/export
    depends_on:
      - db
      - broker
    restart: unless-stopped

  broker:
    image: docker.io/library/redis:7-alpine
    container_name: paperless-ngx_broker
    restart: unless-stopped

  db:
    image: postgres:13
    container_name: paperless-ngx_db
    environment:
      POSTGRES_DB: paperless
      POSTGRES_USER: paperless
      POSTGRES_PASSWORD: paperless
    volumes:
      - paperless_db_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  paperless_data:
  paperless_media:
  paperless_consume:
  paperless_export:
  paperless_db_data:
```