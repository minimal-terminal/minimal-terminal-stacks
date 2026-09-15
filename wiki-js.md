# Wiki.js

Wiki.js is a powerful and extensible open-source Wiki software that makes knowledge management effortless, ideal for centralizing documentation, project notes, and institutional knowledge on your self-hosted server or NAS.

## Prerequisites

*   A Docker-compatible system (e.g., a home server, NAS, or VM).
*   A reverse proxy (like Nginx Proxy Manager, Traefik, or Caddy) is recommended for secure access via HTTPS and custom domains.

## Docker Compose Script

This setup uses PostgreSQL as the database backend. Ensure you replace `your_db_password` with a strong, unique password.

```yaml
version: "3.8"

services:
  wikijs:
    image: ghcr.io/requarks/wiki:2
    container_name: wikijs
    restart: unless-stopped
    ports:
      - "3000:3000" # Wiki.js web interface
    environment:
      - DB_TYPE=postgres
      - DB_HOST=wikijs-db
      - DB_PORT=5432
      - DB_USER=wikijs
      - DB_PASS=your_db_password # <--- CHANGE THIS
      - DB_NAME=wikijs
    volumes:
      - wikijs_data:/wiki/data
    depends_on:
      - wikijs-db

  wikijs-db:
    image: postgres:13-alpine # Use a stable PostgreSQL version
    container_name: wikijs-db
    restart: unless-stopped
    environment:
      - POSTGRES_DB=wikijs
      - POSTGRES_USER=wikijs
      - POSTGRES_PASSWORD=your_db_password # <--- CHANGE THIS
    volumes:
      - wikijs_db:/var/lib/postgresql/data

volumes:
  wikijs_data:
  wikijs_db:
```