# Kanboard

Kanboard is a free and open-source project management software that uses the Kanban methodology. It's a minimalist, powerful, and self-hosted solution for managing tasks and projects efficiently.

## Prerequisites

Before deploying Kanboard with Docker Compose, ensure you have:

*   Docker installed on your system.
*   Docker Compose installed.
*   A suitable host port available (e.g., `8080`) that doesn't conflict with other services.

## Docker Compose Script

This `docker-compose.yml` sets up Kanboard using its official Docker image. It uses a persistent volume to store your project data (SQLite database, configurations, and attachments), ensuring your information is safe across container restarts.

```yaml
version: '3.8'

services:
  kanboard:
    image: kanboard/kanboard:v1.2.31 # Pin to a stable version for consistency
    container_name: kanboard
    ports:
      - "8080:80" # Host_Port:Container_Port. Change 8080 if it conflicts with another service.
    volumes:
      - kanboard_data:/var/www/app/data # Persistent storage for SQLite DB, config, and attachments
    restart: unless-stopped # Ensures the container restarts automatically unless explicitly stopped

volumes:
  kanboard_data:
    driver: local # Defines a local volume for persistent data storage
```

### How to Deploy

1.  Save the content above as `docker-compose.yml` in a directory of your choice.
2.  Open your terminal, navigate to that directory, and run:
    ```bash
    docker compose up -d
    ```
3.  Kanboard will be accessible in your web browser at `http://your-server-ip:8080`.

    The default login credentials are `admin` / `admin`. Remember to change these immediately after your first login for security.