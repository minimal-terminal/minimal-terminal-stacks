```yaml
version: '3.8'

services:
  speedtest-tracker:
    image: ghcr.io/alexjustensen/speedtest-tracker:latest
    container_name: speedtest-tracker
    restart: unless-stopped
    ports:
      - "8080:80" # Expose on port 8080. Change if another port is desired.
    environment:
      - PUID=1000 # User ID, adjust to your host user ID (e.g., `id -u`)
      - PGID=1000 # Group ID, adjust to your host group ID (e.g., `id -g`)
      - TZ=America/New_York # Set your local timezone
      - DB_CONNECTION=sqlite
      - DB_DATABASE=/config/database.sqlite
      # Optional: Set a CRON_SCHEDULE for automatic tests. 
      # Example: "0 * * * *" for every hour, "0 0 * * *" for daily at midnight.
      # If not set, scheduling can be configured from the web UI.
      # - CRON_SCHEDULE="0 * * * *"
    volumes:
      - ./speedtest-tracker/config:/config # Persist configuration and SQLite database
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/api/ping"]
      interval: 1m30s
      timeout: 10s
      retries: 3
      start_period: 30s

```