# Trilium Notes

Trilium Notes is a powerful self-hosted knowledge base application that allows you to create a personal wiki using a hierarchical note structure. It's designed for organizing large bodies of personal knowledge with a rich text editor, code blocks, images, and more, all stored securely on your own server.

## Prerequisites

Before deploying Trilium Notes with Docker Compose, ensure you have:

*   **Docker and Docker Compose:** Installed on your host system (e.g., Linux server, Synology NAS with Docker package).
*   **Data Volume:** A dedicated directory on your host system to persist Trilium Notes data. For this setup, we'll use `./trilium-data` which will be created in the same directory as your `docker-compose.yml` file.
*   **Available Port:** An open port on your host system (e.g., `8080`) to access the Trilium Notes web interface.

## Docker Compose Script

```yaml
version: '3.8'

services:
  trilium:
    image: zadam/trilium:0.62.5
    container_name: trilium
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - ./trilium-data:/home/node/trilium-data
    environment:
      - TZ=America/New_York # Set your desired timezone, e.g., Europe/Berlin
```