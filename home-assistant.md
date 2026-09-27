# Home Assistant: The Ultimate Smart Home Hub

Home Assistant is an open-source home automation platform that puts local control and privacy first. Run it on your own server and integrate thousands of devices and services, from lights and thermostats to sensors and media players, into a single, unified system. Automate your home with powerful scripts and enjoy a truly smart, private, and customizable living experience.

## Prerequisites

Before deploying Home Assistant with Docker Compose, ensure you have:

*   **Docker and Docker Compose:** Installed on your server.
*   **Dedicated Directory:** Create a directory for Home Assistant's configuration files. For example, `mkdir -p ./homeassistant/config`.
*   **Timezone:** Identify your local timezone (e.g., `America/New_York`, `Europe/London`) for the `TZ` environment variable.

## Docker Compose Stack

This `docker-compose.yml` deploys the Home Assistant Core container, exposing its web interface on port `8123`.

```yaml
version: '3.8'

services:
  homeassistant:
    container_name: homeassistant
    image: ghcr.io/home-assistant/home-assistant:stable
    ports:
      - "8123:8123" # Web interface
    volumes:
      - ./config:/config
      - /etc/localtime:/etc/localtime:ro # Optional: Sync host timezone
    environment:
      - TZ=Europe/London # IMPORTANT: Set your timezone here
    restart: unless-stopped
```

## Deployment and Access

1.  **Save the Stack:** Save the content above as `docker-compose.yml` in your chosen directory (e.g., `homeassistant/`).
2.  **Adjust Timezone:** Modify the `TZ` environment variable to match your local timezone.
3.  **Deploy:** Navigate to the directory containing your `docker-compose.yml` and run:
    ```bash
    docker compose up -d
    ```
4.  **Access:** Once the container is running, access the Home Assistant web interface via your server's IP address and port `8123`:
    ```
    http://your-server-ip:8123
    ```

From there, you'll be guided through the initial setup, including creating an administrator account and discovering devices on your network.
