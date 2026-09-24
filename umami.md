# Umami

Umami is a simple, fast, privacy-focused alternative to Google Analytics. It allows you to track website visitors with full data ownership, providing a clean and intuitive dashboard for essential insights without compromising user privacy.

## Prerequisites

To run Umami with Docker Compose, you will need:
- Docker installed on your system.
- Docker Compose installed on your system.
- Basic understanding of environment variables and volume mapping.

## Docker Compose Script

Create a `docker-compose.yml` file in a directory of your choice, along with a `postgres-data` subdirectory for persistent database storage.

Remember to replace `your_strong_password` and `your_super_secret_app_key` with strong, randomly generated values. The `APP_SECRET` is critical for security and session management.

```yaml
version: '3.8'

services:
  umami:
    image: umami/umami:latest
    container_name: umami
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgresql://umami:your_strong_password@db:5432/umami
      APP_SECRET: your_super_secret_app_key # REQUIRED: Generate a strong, random key
    depends_on:
      - db
    networks:
      - umami-network

  db:
    image: postgres:14-alpine
    container_name: umami_db
    restart: unless-stopped
    environment:
      POSTGRES_DB: umami
      POSTGRES_USER: umami
      POSTGRES_PASSWORD: your_strong_password # REQUIRED: Use the same password as above
    volumes:
      - ./postgres-data:/var/lib/postgresql/data
    networks:
      - umami-network

networks:
  umami-network:
    driver: bridge
```