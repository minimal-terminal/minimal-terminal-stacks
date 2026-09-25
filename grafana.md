```markdown
# Grafana: The Observability Platform

Grafana is an open-source analytics and interactive visualization web application. It allows you to query, visualize, alert on, and explore your metrics, logs, and traces no matter where they are stored. It's a fundamental tool for any homelab or production environment, providing unparalleled insight into your systems' performance and health.

## Prerequisites

Before deploying Grafana, ensure you have:

*   **Docker and Docker Compose:** Installed on your host machine (e.g., Linux server, NAS).
*   **Persistent Storage:** A directory or Docker volume to store Grafana's configuration and data.
*   **.env file (Recommended):** Create a `.env` file in the same directory as your `docker-compose.yml` to manage sensitive environment variables securely.

    ```dotenv
    # .env file for Grafana
    GF_SECURITY_ADMIN_USER=admin
    GF_SECURITY_ADMIN_PASSWORD=your_secure_password
    ```

    *Replace `your_secure_password` with a strong, unique password.*

## Docker Compose Stack

This `docker-compose.yml` sets up a basic Grafana instance with persistent storage and environment variables for initial admin credentials.

```yaml
version: '3.8'

services:
  grafana:
    image: grafana/grafana-oss:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3000:3000" # Host_Port:Container_Port
    volumes:
      - grafana_data:/var/lib/grafana
      # Optional: Mount a custom configuration directory if needed
      # - ./grafana/config:/etc/grafana
    environment:
      - GF_SECURITY_ADMIN_USER=${GF_SECURITY_ADMIN_USER}
      - GF_SECURITY_ADMIN_PASSWORD=${GF_SECURITY_ADMIN_PASSWORD}
      # Uncomment and adjust if Grafana is behind a reverse proxy
      # - GF_SERVER_ROOT_URL=https://grafana.yourdomain.com
      # - GF_SERVER_SERVE_FROM_SUB_PATH=true
    networks:
      - grafana_network

volumes:
  grafana_data:
    driver: local

networks:
  grafana_network:
    driver: bridge
```

## Deployment Steps

1.  **Save the Compose File:** Save the content above as `docker-compose.yml` in a directory of your choice (e.g., `~/docker/grafana`).
2.  **Create .env File:** Create a `.env` file in the *same directory* as `docker-compose.yml` and populate it with your desired `GF_SECURITY_ADMIN_USER` and `GF_SECURITY_ADMIN_PASSWORD`.
3.  **Deploy the Stack:** Open your terminal, navigate to the directory containing your `docker-compose.yml` and `.env` files, and run:
    ```bash
    docker compose up -d
    ```
    (Use `docker-compose` instead of `docker compose` for older Docker versions).

## Accessing Grafana

Once the container is running, Grafana will be accessible via your web browser at `http://your_server_ip:3000`.

Log in with the username (`admin`) and password you defined in your `.env` file (`your_secure_password`). You will be prompted to change the default admin password on your first login, which is highly recommended for security.

From here, you can start adding data sources (e.g., Prometheus, InfluxDB, MySQL) and building your custom dashboards to monitor your homelab infrastructure.
