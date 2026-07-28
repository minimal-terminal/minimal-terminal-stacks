# Dashy

Dashy is a highly customizable, self-hosted dashboard for your homelab, allowing you to organize all your applications, services, and links in one beautiful interface. It's open-source, privacy-respecting, and features a wide array of customization options, including themes, widgets, and quick-launch capabilities.

## Prerequisites

Ensure you have a Docker volume mapped for persistent configuration and data. Choose an available port on your host system to expose Dashy's web interface.

## Docker Compose Script

```yaml
version: '3.8'
services:
  dashy:
    image: lissy93/dashy:latest
    container_name: dashy
    ports:
      - "8080:80" # Host_Port:Container_Port - Change 8080 to your desired port
    volumes:
      - ./dashy/conf.yml:/app/public/conf.yml # Mount your config file
      - ./dashy/data:/app/data # Optional: For persistent data like custom images, etc.
    environment:
      # Optional: Set a custom base URL if running behind a reverse proxy sub-path
      # - BASE_URL=/dashy
      - NODE_ENV=production
    restart: unless-stopped
    # Optional: If you want to use a specific user/group ID for volume permissions
    # user: "1000:1000"
```