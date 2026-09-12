# Traefik

Traefik is a modern HTTP reverse proxy and load balancer designed to deploy microservices with ease. It integrates directly with your existing infrastructure components (like Docker) and automatically configures itself. Traefik provides automatic SSL certificate management (via Let's Encrypt), making it simple to secure your applications with HTTPS.

## Prerequisites

Before deploying Traefik, ensure you have:

*   A Docker-compatible environment (e.g., a home server, NAS with Docker support).
*   A domain name pointed to your server's public IP address (for Let's Encrypt).
*   Ports 80 and 443 forwarded to your server if accessing services from outside your local network.

## Docker Compose Script

```yaml
version: '3.8'

services:
  traefik:
    image: traefik:v2.10 # Use a stable v2.x tag
    container_name: traefik
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    ports:
      - "80:80"   # The HTTP port
      - "443:443" # The HTTPS port
      # - "8080:8080" # The Web UI (Dashboard) port - uncomment to expose for direct access
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro # So Traefik can listen to Docker events
      - ./traefik.yml:/etc/traefik/traefik.yml:ro    # Main Traefik configuration file
      - ./acme.json:/acme.json                       # Certificate storage for Let's Encrypt
    networks:
      - traefik_proxy # Connect to a dedicated proxy network
    environment:
      # Optional: Set PUID and PGID for file permissions if needed for acme.json ownership
      # - PUID=1000
      # - PGID=1000
      # Recommended: Set your timezone
      - TZ=America/New_York # Replace with your timezone, e.g., Europe/London

networks:
  traefik_proxy:
    external: true # Assumes you've created this network: `docker network create traefik_proxy`
```

### Configuration Files

#### `traefik.yml`

Create a file named `traefik.yml` in the same directory as your `docker-compose.yml` with the following content. Replace `your.domain.com` and `your-email@example.com` with your actual domain and email.

```yaml
# traefik.yml
api:
  dashboard: true
  insecure: true # Don't use in production, only for initial dashboard access via 8080

entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: "websecure"
          scheme: "https"
          permanent: true
  websecure:
    address: ":443"

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false # Only expose containers that have traefik.enable=true label
    network: traefik_proxy # Specify the network Traefik should listen on for Docker containers
  file:
    directory: /etc/traefik/dynamic/
    watch: true # Watch for changes in the dynamic configuration directory

certificatesResolvers:
  letsencrypt:
    acme:
      email: "your-email@example.com" # Replace with your email
      storage: "acme.json"
      # Use the HTTP-01 challenge for easier setup
      httpChallenge:
        entryPoint: "web"
      # For production, use the production ACME endpoint:
      # caServer: "https://acme-v02.api.letsencrypt.org/directory"
      # For testing, use the staging ACME endpoint (rate limit friendly):
      caServer: "https://acme-staging-v02.api.letsencrypt.org/directory"

log:
  level: INFO
accessLog: {}
```

#### `acme.json`

Create an empty file named `acme.json` in the same directory as your `docker-compose.yml`. This file will store your Let's Encrypt certificates. **Crucially, set its permissions to 600** so only the owner can read/write it.

```bash
touch acme.json
chmod 600 acme.json
```

### Usage

1.  **Create the Traefik network:**
    ```bash
    docker network create traefik_proxy
    ```
2.  **Create the `acme.json` file and set permissions:**
    ```bash
    touch acme.json
    chmod 600 acme.json
    ```
3.  **Create the `traefik.yml` file** as described above, replacing placeholders.
4.  **Deploy Traefik:**
    ```bash
    docker compose up -d
    ```

### Exposing Services

To expose a service through Traefik, add labels to its `docker-compose.yml` service definition. Here's an example for a simple Whoami app:

```yaml
version: '3.8'

services:
  whoami:
    image: traefik/whoami
    container_name: whoami
    networks:
      - traefik_proxy # Connect to the Traefik proxy network
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.whoami.rule=Host(`whoami.your.domain.com`)" # Replace with your subdomain
      - "traefik.http.routers.whoami.entrypoint=websecure"
      - "traefik.http.routers.whoami.tls.certresolver=letsencrypt"
      - "traefik.http.services.whoami.loadbalancer.server.port=80" # Default port for whoami

networks:
  traefik_proxy:
    external: true
```

After deploying Traefik, deploy your `whoami` service. Traefik will automatically detect it, configure routing, and obtain an SSL certificate for `whoami.your.domain.com`.