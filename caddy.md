# Caddy

Caddy is an open-source web server with automatic HTTPS. It is a powerful, enterprise-ready web server that is easy to use. Caddy supports HTTP/1.1, HTTP/2, and HTTP/3 and can act as a reverse proxy, load balancer, and API gateway. Its primary advantage is its automatic HTTPS functionality, which obtains and renews SSL/TLS certificates for your domains with zero configuration.

## Prerequisites

Before deploying Caddy, ensure you have:

*   A domain name pointing to your server's public IP address.
*   Ports 80 and 443 forwarded from your router to your server's internal IP address where Caddy will run.
*   A `Caddyfile` configuration in the same directory as your `docker-compose.yml` to define your reverse proxy rules and web server configurations.

## Docker Compose Script

```yaml
version: "3.8"

services:
  caddy:
    image: caddy:2.7.6-alpine # Using a stable Alpine-based image for efficiency
    container_name: caddy
    restart: unless-stopped
    ports:
      - "80:80" # HTTP traffic
      - "443:443" # HTTPS traffic, required for automatic certificates
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile # Mount your Caddy configuration file
      - ./caddy_data:/data # Persistent data for certificates and state
      - ./caddy_config:/config # Persistent configuration
    environment:
      - PUID=1000 # Your user ID, adjust as needed for file permissions
      - PGID=1000 # Your group ID, adjust as needed for file permissions
      - TZ=Europe/London # Your timezone, e.g., America/New_York
    networks:
      - caddy_network

networks:
  caddy_network:
    # Define a custom bridge network for Caddy to communicate with other services.
    # If other self-hosted apps are on different networks, you might need to connect
    # Caddy to those specific networks as well, or place them all on this network.
    driver: bridge
```

### Example Caddyfile

Create a file named `Caddyfile` in the same directory as your `docker-compose.yml` with content similar to this:

```caddyfile
# Example Caddyfile for a simple reverse proxy
# Replace 'your.domain.com' with your actual domain
# Replace 'your_app_container_name:port' with the internal Docker service name and port

your.domain.com {
  reverse_proxy your_app_container_name:port
  # Example for a local service on the same Docker network:
  # reverse_proxy my-nextcloud:80

  # Optional: Enable Caddy's access log
  # log {
  #   output stdout
  #   format json
  # }
}

# You can add multiple sites or configurations:
# another.domain.com {
#   reverse_proxy another_app:8080
# }

# Example for serving static files directly
# static.domain.com {
#   root * /srv # Assuming '/srv' directory is mounted into Caddy container
#   file_server
# }
```