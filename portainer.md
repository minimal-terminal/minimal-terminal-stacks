# Portainer

Portainer is an open-source tool that provides a graphical user interface (GUI) for managing Docker environments. It simplifies the deployment, management, and monitoring of Docker containers, images, volumes, and networks, making it an invaluable asset for anyone running Docker on a home server or NAS.

## Prerequisites

Before deploying Portainer, ensure you have Docker and Docker Compose installed on your system. Portainer requires access to the Docker socket to manage containers.

## Docker Compose Script

Here's a `docker-compose.yml` script for deploying Portainer CE (Community Edition). This setup ensures Portainer data is persistent and accessible via port `9000`.

```yaml
version: '3.9'

services:
  portainer:
    image: portainer/portainer-ce:2.19.4 # Use a specific stable version for production
    container_name: portainer
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock # Mount the Docker socket for management
      - portainer_data:/data # Persistent storage for Portainer's data
    ports:
      - "9000:9000" # Web UI for Portainer
      # - "8000:8000" # Optional: For Edge agent communication, not needed for basic setup
    environment:
      # Optional: Set timezone for logs if needed
      - TZ=America/New_York # Example: Set your timezone
    networks:
      - default # Or a custom bridge network if you have one

volumes:
  portainer_data:

networks:
  default:
    # Use an external network if you have one for your services, e.g., for Traefik/NPM
    # external: true
    # name: my-custom-bridge-network
```

**To deploy:**

1.  Save the content above as `docker-compose.yml` in a directory (e.g., `/opt/portainer`).
2.  Navigate to that directory in your terminal.
3.  Run `docker compose up -d`.

After deployment, Portainer will be accessible via your server's IP address on port `9000` (e.g., `http://your-server-ip:9000`). The first time you access it, you'll be prompted to create an admin user.