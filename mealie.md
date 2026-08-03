# Mealie

Mealie is a self-hosted recipe manager and meal planner that allows you to store, organize, and plan your culinary adventures. It features a modern interface, powerful search, meal planning capabilities, and easy recipe importing from various websites.

## Prerequisites

Ensure you have Docker and Docker Compose installed.
You will need to create a directory for Mealie's data persistence, e.g., `/your/path/to/mealie/data`.
Mealie requires a PostgreSQL database. The provided Docker Compose sets up a dedicated PostgreSQL container.

## Docker Compose Script

```yaml
version: '3.8'

services:
  mealie:
    image: hkotel/mealie:latest
    container_name: mealie
    restart: unless-stopped
    ports:
      - "9000:80" # Mealie web interface
    environment:
      # General Mealie settings
      - ALLOW_SIGNUP=true # Set to false after initial user creation for security
      - PUID=1000 # User ID (e.g., your user ID)
      - PGID=1000 # Group ID (e.g., your group ID)
      - TZ=Etc/UTC # Your timezone, e.g., America/New_York

      # Database connection settings (referencing the 'db' service)
      - DB_ENGINE=postgresql
      - POSTGRES_SERVER=db
      - POSTGRES_PORT=5432
      - POSTGRES_USER=mealie
      - POSTGRES_PASSWORD=your_db_password # CRITICAL: Change this to a strong password
      - POSTGRES_DB=mealie

    volumes:
      - /your/path/to/mealie/data:/app/data # CRITICAL: Change to your host path
      - /etc/localtime:/etc/localtime:ro # Optional: Sync host timezone

  db:
    image: postgres:13-alpine # Using a specific version for stability
    container_name: mealie_db
    restart: unless-stopped
    environment:
      - POSTGRES_DB=mealie
      - POSTGRES_USER=mealie
      - POSTGRES_PASSWORD=your_db_password # CRITICAL: Match the password in Mealie service
    volumes:
      - /your/path/to/mealie/db:/var/lib/postgresql/data # CRITICAL: Change to your host path for DB data
    healthcheck: # Ensure DB is ready before Mealie starts
      test: ["CMD-SHELL", "pg_isready -U mealie"]
      interval: 5s
      timeout: 5s
      retries: 5
```