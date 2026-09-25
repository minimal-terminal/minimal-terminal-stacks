### Prerequisites

Before deploying Gotify, ensure you have Docker and Docker Compose installed on your system. Create a dedicated directory for Gotify to store its configuration and data.

### Docker Compose Stack

This `docker-compose.yml` deploys the Gotify server, providing a robust, self-hosted push notification service.

```yaml
version: '3.8'

services:
  gotify:
    image: gotify/server:latest
    container_name: gotify
    restart: unless-stopped
    ports:
      # Map container port 80 to host port 8080.
      # Adjust '8080' if it conflicts with other services.
      # For production, consider placing Gotify behind a reverse proxy (like Caddy, Nginx Proxy Manager, or Traefik)
      # and exposing only port 80/443 on the proxy, then routing to Gotify's internal port 80.
      - "8080:80"
    volumes:
      # Persist Gotify's data in a named volume or a bind mount.
      # This example uses a bind mount to a 'data' subdirectory.
      - ./data:/app/data
    environment:
      # Set initial admin user credentials.
      # **CRITICAL:** Change 'your_strong_admin_password' to a secure, unique password.
      # You can manage users and applications from the Gotify UI after initial setup.
      - GOTIFY_DEFAULTUSER_NAME=admin
      - GOTIFY_DEFAULTUSER_PASS=your_strong_admin_password
      # For advanced users: Configure an external database (PostgreSQL, MySQL) instead of the default SQLite.
      # Example for PostgreSQL:
      # - GOTIFY_SERVER_DATABASE_DIALECT=postgres
      # - GOTIFY_SERVER_DATABASE_CONNECTION=postgres://user:password@host:5432/gotify?sslmode=disable
```

### Deployment Instructions

1.  **Save the file:** Save the content above as `docker-compose.yml` in your chosen directory (e.g., `~/docker/gotify/`).
2.  **Create data directory:** Inside the same directory, create a `data` subdirectory: `mkdir data`.
3.  **Deploy:** Open your terminal, navigate to the directory where you saved `docker-compose.yml`, and run:
    ```bash
    docker compose up -d
    ```
4.  **Access:** Once deployed, Gotify will be accessible via your server's IP address and the specified host port (e.g., `http://your-server-ip:8080`). Log in with the `admin` user and the password you set.

### Post-Deployment

*   **Security:** Immediately change the default admin password via the Gotify UI.
*   **Applications:** Create "applications" within Gotify to generate unique tokens for different services or scripts that will send notifications.
*   **Clients:** Download Gotify clients for your mobile devices (Android, iOS) or use web clients to receive push notifications.
*   **Reverse Proxy:** For secure, external access and custom domains, strongly consider putting Gotify behind a reverse proxy with SSL/TLS encryption.