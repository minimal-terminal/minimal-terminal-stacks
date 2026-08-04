# Vikunja

Vikunja is a free, open-source, and self-hostable task management system. It helps you organize your tasks, projects, and goals across multiple platforms with a clean, intuitive interface. Vikunja is designed for both personal use and small teams, offering powerful features like boards, lists, sub-tasks, and integrations with other services.

## Prerequisites

Before deploying Vikunja, ensure you have:
*   Docker and Docker Compose installed.
*   A dedicated directory for Vikunja's configuration and data (e.g., `/your/appdata/vikunja`).
*   Open ports (e.g., `3000` for the API and `8080` for the frontend if exposed directly, or via a reverse proxy).
*   A PostgreSQL database is recommended for production use. This script includes a PostgreSQL service.

## Docker Compose Script

```yaml
version: "3.8"

services:
  vikunja-db:
    image: postgres:15-alpine
    container_name: vikunja-db
    restart: unless-stopped
    environment:
      POSTGRES_DB: vikunja
      POSTGRES_USER: vikunja
      POSTGRES_PASSWORD: your_strong_db_password # <--- CHANGE THIS
    volumes:
      - /your/appdata/vikunja/db:/var/lib/postgresql/data

  vikunja-api:
    image: vikunja/api
    container_name: vikunja-api
    restart: unless-stopped
    environment:
      VIKUNJA_DATABASE_TYPE: postgres
      VIKUNJA_DATABASE_HOST: vikunja-db:5432
      VIKUNJA_DATABASE_USER: vikunja
      VIKUNJA_DATABASE_PASSWORD: your_strong_db_password # <--- CHANGE THIS, MUST MATCH DB
      VIKUNJA_DATABASE_DATABASE: vikunja
      VIKUNJA_SERVICE_FRONTENDURL: http://your_domain_or_ip:8080 # <--- CHANGE THIS to your frontend's publicly accessible URL (e.g., http://tasks.yourdomain.com). Required for OAuth/email links.
      VIKUNJA_SERVICE_ENABLEEMAILREMINDERS: "false" # Set to "true" and configure SMTP for email reminders
      VIKUNJA_JWT_SECRET: your_jwt_secret_key # <--- CHANGE THIS to a random, long string
      VIKUNJA_OAUTH_OAUTH2CLIENTS_GITHUB_ENABLED: "false" # Example: enable/disable OAuth providers
      VIKUNJA_OAUTH_OAUTH2CLIENTS_GOOGLE_ENABLED: "false"
    volumes:
      - /your/appdata/vikunja/config:/app/vikunja/config
      - /your/appdata/vikunja/files:/app/vikunja/files # For attachments
    depends_on:
      - vikunja-db
    ports:
      - "3000:3000" # API port. Only expose if accessing directly; otherwise, use a reverse proxy (recommended).

  vikunja-frontend:
    image: vikunja/frontend
    container_name: vikunja-frontend
    restart: unless-stopped
    environment:
      VIKUNJA_API_URL: http://your_domain_or_ip:3000/api/v1 # <--- CHANGE THIS to your API's publicly accessible URL (e.g., http://api.tasks.yourdomain.com/api/v1)
    ports:
      - "8080:80" # Frontend port. Change if 8080 is in use, or use a reverse proxy (recommended).
    depends_on:
      - vikunja-api
```