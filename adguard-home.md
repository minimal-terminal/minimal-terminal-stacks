# AdGuard Home

AdGuard Home is a network-wide software for blocking ads & tracking. After you set it up, it'll cover ALL your devices, and you won't need any client-side software. It acts as a DNS server, re-routing ad and tracker domains to a 'blackhole' to prevent them from loading.

## Prerequisites

Before deploying AdGuard Home, ensure you have a standard Docker installation. The following ports must be available on your host machine:

*   **53 (TCP/UDP)**: Standard DNS queries.
*   **80 (TCP)**: Initial web interface setup and HTTP access (can be changed later).
*   **443 (TCP/UDP)**: HTTPS for web interface (if configured) and DNS-over-HTTPS/TLS.
*   **3000 (TCP)**: Alternative web interface port (if 80/443 are already in use).
*   **853 (TCP)**: DNS-over-TLS.

You will also need to create a directory for AdGuard Home's configuration and working data, e.g., `./adguard` relative to your `docker-compose.yml` file.

## Docker Compose Script

```yaml
version: '3.8'

services:
  adguardhome:
    image: adguard/adguardhome:latest
    container_name: adguardhome
    ports:
      - '53:53/tcp' # DNS
      - '53:53/udp' # DNS
      - '80:80/tcp' # Initial setup UI, can be changed later
      - '443:443/tcp' # HTTPS UI / DNS-over-HTTPS
      - '443:443/udp' # DNS-over-QUIC
      - '3000:3000/tcp' # Alternative UI port
      - '853:853/tcp' # DNS-over-TLS
    volumes:
      - ./adguard/work:/opt/adguardhome/work
      - ./adguard/conf:/opt/adguardhome/conf
    restart: unless-stopped
    # Uncomment the following line if you want to use the host network directly.
    # This simplifies DNS setup but might have security implications.
    # network_mode: host

networks:
  default:
    driver: bridge
```