# FreshRSS

FreshRSS is a free, self-hostable RSS feed aggregator. It is a lightweight, fast, and powerful solution allowing you to follow all your favorite websites without being tracked.

## Prerequisites

Before deploying FreshRSS, ensure you have:

*   Docker installed on your system.
*   Docker Compose installed.
*   A designated directory on your host machine for persistent FreshRSS data (e.g., `/path/to/your/freshrss/data`).

## Docker Compose Script

```yaml
version: '3.8'

services:
  freshrss:
    image: freshrss/freshrss:latest
    container_name: freshrss
    restart: unless-stopped
    ports:
      - "8080:80" # Host port:Container port - Change 8080 if it conflicts with another service
    volumes:
      - /path/to/your/freshrss/data:/var/www/FreshRSS/data # Persistent storage for application data
    environment:
      - TZ=Etc/UTC # Set your desired timezone, e.g., America/New_York
      # For more advanced configurations (e.g., external database, PUID/PGID),
      # refer to the official FreshRSS Docker documentation.
```

**To deploy FreshRSS:**

1.  Save the above content as `docker-compose.yml` in your chosen directory.
2.  Replace `/path/to/your/freshrss/data` with the actual path on your host machine where you want to store FreshRSS's persistent data.
3.  Adjust the `8080` host port if it conflicts with another service running on your machine.
4.  Update the `TZ` environment variable to your local timezone.
5.  Open your terminal, navigate to the directory where you saved `docker-compose.yml`, and run:
    ```bash
    docker compose up -d
    ```
6.  Access FreshRSS via your web browser at `http://your-server-ip:8080` (or the port you configured).

FreshRSS uses SQLite by default, which is sufficient for most home users. For larger installations or specific needs, FreshRSS supports external databases like MySQL/MariaDB or PostgreSQL, which would require additional services in your `docker-compose.yml`.