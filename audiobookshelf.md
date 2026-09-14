# Audiobookshelf

Audiobookshelf is a self-hosted audiobook and podcast server for managing and streaming your digital audio library.

## Prerequisites

Before deploying Audiobookshelf with Docker Compose, ensure you have:

*   Docker and Docker Compose installed on your host system.
*   A dedicated directory for Audiobookshelf's configuration data (e.g., `./config`).
*   A dedicated directory for your audiobook files (e.g., `./audiobooks`).
*   (Optional) A dedicated directory for podcast downloads (e.g., `./podcasts`).
*   The `PUID` and `PGID` of the user on your host system that Docker will run as. This ensures proper file permissions for your library. You can typically find these using `id -u yourusername` and `id -g yourusername`.

## Docker Compose Script

Here's a production-ready `docker-compose.yml` for Audiobookshelf:

```yaml
version: '3.8'
services:
  audiobookshelf:
    image: ghcr.io/advplyr/audiobookshelf:latest
    container_name: audiobookshelf
    restart: unless-stopped
    ports:
      - "6800:6800" # Web UI and streaming port
    volumes:
      - ./config:/config # Persistent storage for Audiobookshelf configuration and metadata
      - ./audiobooks:/audiobooks # Mount your local directory containing audiobook files
      - ./podcasts:/podcasts # Optional: Mount for storing downloaded podcast episodes
    environment:
      - PUID=1000 # User ID for container processes (adjust to your host user's UID)
      - PGID=1000 # Group ID for container processes (adjust to your host user's GID)
      - TZ=Etc/UTC # Set your desired timezone, e.g., America/New_York
```

**To deploy:**

1.  Save the content above as `docker-compose.yml` in a new directory (e.g., `audiobookshelf`).
2.  Create the `config`, `audiobooks`, and `podcasts` (optional) subdirectories within this new directory.
3.  Place your audiobook and podcast files into the respective mounted directories.
4.  Adjust `PUID`, `PGID`, and `TZ` environment variables to match your system and preferences.
5.  Open your terminal in the directory where `docker-compose.yml` is located and run: `docker-compose up -d`

Audiobookshelf will then be accessible via your browser at `http://your-server-ip:6800`.