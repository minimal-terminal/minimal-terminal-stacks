# PhotoPrism

PhotoPrism is an AI-powered photos app for browsing, organizing, and sharing your personal photo and video collection. It makes use of machine learning to automatically tag and find pictures based on their content and location, making your photo library searchable and beautiful.

## Prerequisites

Before deploying PhotoPrism, ensure you have:

*   **Docker and Docker Compose:** Installed on your host system (e.g., Synology NAS, Ubuntu server).
*   **Data Volumes:** Dedicated Docker volumes for PhotoPrism's internal data, your original photo files, and its cache. These will ensure data persistence and allow easy management of your media.
    *   `photoprism_data`: Stores PhotoPrism's database, index, and configuration.
    *   `photoprism_photos`: This is where you'll mount your existing photo and video library. PhotoPrism will read from here.
    *   `photoprism_cache`: Stores generated thumbnails, sidecar files, and other cached data.
*   **Port Availability:** Ensure port `2342` is available on your host for PhotoPrism's web interface.

## Docker Compose Script

Save the following content as `docker-compose.yml` and deploy using `docker compose up -d`.

```yaml
version: '3.8'

services:
  photoprism:
    image: photoprism/photoprism:latest
    container_name: photoprism
    restart: unless-stopped
    ports:
      - "2342:2342" # Access PhotoPrism via http://your-server-ip:2342
    environment:
      # Admin user and password - CRITICAL: CHANGE THESE DEFAULT VALUES!
      - PHOTOPRISM_ADMIN_USER=admin
      - PHOTOPRISM_ADMIN_PASSWORD=please-change-me-strong-password
      
      # General Configuration
      - PHOTOPRISM_HTTP_PORT=2342
      - TZ=America/New_York # Set your timezone, e.g., Europe/Berlin
      - PUID=1000 # Optional: User ID for file permissions, adjust to your host user
      - PGID=1000 # Optional: Group ID for file permissions, adjust to your host group
      
      # Advanced Features (uncomment/adjust as needed)
      - PHOTOPRISM_DATABASE_DRIVER=sqlite # Use 'mysql' or 'mariadb' for external databases
      # - PHOTOPRISM_DETECT_NSFW=false # Set to 'true' to enable NSFW detection (requires additional models)
      # - PHOTOPRISM_DISABLE_WEBDAV=false # Set to 'true' to disable WebDAV access
      # - PHOTOPRISM_EXPERIMENTAL=false # Set to 'true' to enable experimental features

    volumes:
      # PhotoPrism's internal data (database, config, index)
      - photoprism_data:/photoprism/storage
      # Mount your existing photo/video library here. PhotoPrism reads from this.
      # Replace /path/to/your/photos with the actual path on your host.
      - photoprism_photos:/photoprism/photos
      # PhotoPrism's cache for thumbnails, transcoded videos, etc.
      - photoprism_cache:/photoprism/cache

volumes:
  photoprism_data:
  photoprism_photos:
  photoprism_cache:
```