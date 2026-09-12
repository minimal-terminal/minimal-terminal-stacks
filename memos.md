# Memos

Memos is a lightweight, open-source, self-hosted knowledge base designed for capturing and managing your daily thoughts, notes, and ideas with simplicity and efficiency. It offers a clean interface, Markdown support, and powerful tagging for easy organization and retrieval of your personal knowledge.

## Prerequisites

Before deploying Memos with Docker Compose, ensure you have:

-   Docker and Docker Compose installed on your system.
-   A dedicated directory for Memos' persistent data (e.g., `/your/appdata/path/memos`).
-   An available port (default is `5230`) on your host machine.

## Docker Compose Script

```yaml
version: '3.8'
services:
  memos:
    image: ghcr.io/usememos/memos:latest
    container_name: memos
    restart: unless-stopped
    ports:
      - "5230:5230"
    volumes:
      - /your/appdata/path/memos:/var/opt/memos
    environment:
      # Optional: Set PUID and PGID if running on a system where
      # container user/group permissions need to match host user/group.
      # For Memos, default permissions are usually fine, but you can
      # uncomment and adjust these if you encounter permission issues.
      # - PUID=1000
      # - PGID=1000
```