# Filebrowser

Filebrowser is a lightweight, web-based file manager that allows you to browse, upload, download, and manage files on your server or NAS through a clean and intuitive interface. It's perfect for quickly accessing and organizing your self-hosted data from any web browser.

## Prerequisites

*   Docker and Docker Compose installed on your host system.
*   A dedicated directory on your host for Filebrowser's configuration and database (e.g., `/opt/filebrowser/config`).
*   A directory on your host containing the files you wish to manage (e.g., `/mnt/user/data/shared_files`).

## Docker Compose Script

Replace `/path/to/your/filebrowser_config` and `/path/to/your/shared_files` with your actual host paths. Adjust the host port `8000` if it conflicts with other services. For optimal permissions, consider replacing `1000:1000` with the UID/GID of your host user (e.g., `id -u` and `id -g`).

```yaml
version: '3.8'

services:
  filebrowser:
    image: filebrowser/filebrowser:latest
    container_name: filebrowser
    restart: unless-stopped
    ports:
      - "8000:80" # Host_Port:Container_Port - Access Filebrowser via http://your-server-ip:8000
    volumes:
      - /path/to/your/filebrowser_config:/config # Persistent storage for Filebrowser's configuration and database
      - /path/to/your/shared_files:/srv # Mount your files here. "/srv" is the default root for Filebrowser.
    environment:
      # Optionally specify the database path within the config volume for clarity
      - FB_DATABASE=/config/database.db
    user: "1000:1000" # Recommend using your host user/group ID for proper file permissions.
                      # Find your UID/GID with 'id -u' and 'id -g' on your host.
```