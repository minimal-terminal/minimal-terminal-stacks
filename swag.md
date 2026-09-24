# SWAG (Secure Web Application Gateway)

SWAG, formerly known as LinuxServer.io's Nginx with Let's Encrypt, is a powerful and versatile Docker image that acts as a secure web application gateway. It combines Nginx as a reverse proxy with an automated Let's Encrypt client to provide free SSL/TLS certificates, ensuring secure, encrypted access to your self-hosted services from the internet.

## Prerequisites

Before deploying SWAG, ensure you have:

*   **A Registered Domain Name:** You'll need a domain that you own (e.g., `example.com`).
*   **DNS Configuration:** Create `A` or `CNAME` records with your domain registrar or DNS provider that point your domain and any desired subdomains (e.g., `www.example.com`, `cloud.example.com`) to the public IP address of the server running SWAG.
*   **Port Forwarding:** For HTTP validation (the default in this script), ensure ports `80` (HTTP) and `443` (HTTPS) are forwarded from your router to the internal IP address of your Docker host. If using DNS validation, you do not need to forward port `80` or `443` externally.
*   **User IDs (PUID/PGID):** Determine the `PUID` and `PGID` for the user you want SWAG to run as (e.g., `1000` for the default user on many Linux systems).

## Docker Compose Script

```yaml
version: "3.8"
services:
  swag:
    image: lscr.io/linuxserver/swag:latest
    container_name: swag
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1000 # Your User ID
      - PGID=1000 # Your Group ID
      - TZ=Etc/UTC # Your Timezone, e.g., America/New_York
      - URL=yourdomain.com # Your primary domain, e.g., example.com
      - SUBDOMAINS=www, # Comma-separated list of subdomains, e.g., www,cloud,plex
      - VALIDATION=http # Use http for port 80 validation, or dns for DNS challenge
      - EMAIL=your-email@example.com # Email for Let's Encrypt notifications
      # - DNSPLUGIN=cloudflare # Uncomment and specify if using DNS validation (e.g., cloudflare, duckdns, godaddy)
      # - DUCKDNSTOKEN=YOUR_DUCKDNS_TOKEN # Uncomment if using duckdns validation
      # - CF_API_EMAIL=your-cf-email@example.com # Uncomment if using cloudflare validation
      # - CF_API_KEY=YOUR_CF_API_KEY # Uncomment if using cloudflare validation
      - ONLY_SUBDOMAINS=false # Set to true to only generate certs for subdomains, not the root domain
      # - STAGING=true # Uncomment for Let's Encrypt staging environment (for testing, avoids rate limits)
    volumes:
      - /path/to/your/swag/config:/config # Host path for SWAG configuration and certificates
    ports:
      - 443:443
      - 80:80
    restart: unless-stopped
    networks:
      - internal_network # Connect to a custom Docker network for internal services

networks:
  internal_network:
    external: true # Use an existing external network for your self-hosted apps
    # If you don't have an external network, remove 'external: true' and uncomment the line below
    # name: internal_network
```

**Explanation of Variables and Paths:**

*   `PUID`/`PGID`: Set these to match the user and group IDs on your host system that will own the files. You can find them using `id -u youruser` and `id -g youruser`.
*   `TZ`: Set your local timezone.
*   `URL`: Replace `yourdomain.com` with your actual primary domain.
*   `SUBDOMAINS`: A comma-separated list of subdomains you want certificates for. For example, if you want `www.yourdomain.com` and `cloud.yourdomain.com`, set `SUBDOMAINS=www,cloud`. Leave empty if you only want the root domain.
*   `VALIDATION`: `http` is common and uses port 80. `dns` is an alternative that requires a `DNSPLUGIN` and corresponding API credentials, useful if port 80/443 aren't directly accessible.
*   `EMAIL`: Your email address for Let's Encrypt renewal notifications.
*   `ONLY_SUBDOMAINS`: If `true`, SWAG will only generate certificates for the `SUBDOMAINS` specified, not the root `URL`.
*   `STAGING`: Set to `true` for testing certificate issuance without hitting Let's Encrypt's rate limits. Remember to set it back to `false` for production certificates.
*   `/path/to/your/swag/config`: Change this to the absolute path on your host where you want SWAG's configuration files and SSL certificates to be stored persistently.
*   `networks`: It's highly recommended to connect SWAG to a custom Docker network that your other self-hosted applications also use. This allows SWAG to act as a reverse proxy for them by referring to their container names within that network. If `internal_network` doesn't exist, create it with `docker network create internal_network` or modify the `networks` section to define it internally.

After starting SWAG, you can configure your individual service proxy configurations by creating new `.conf` files in the `/config/nginx/proxy-confs/` directory within your SWAG volume. Example proxy configurations for many popular self-hosted apps are often included in the `/config/nginx/proxy-confs/` directory as `.sample` files.