# Pi-hole

Pi-hole acts as a DNS sinkhole, protecting your network from unwanted content like ads and trackers without the need for client-side software. By routing all DNS queries through Pi-hole, you gain network-wide protection, detailed statistics, and customizable blocking lists. It's an essential tool for a cleaner, faster, and more secure browsing experience on all your devices.

## Prerequisites

Before deploying Pi-hole with Docker Compose, ensure you have:

*   **Docker and Docker Compose** installed on your host system (e.g., Linux server, Synology NAS).
*   **Persistent Storage:** A dedicated directory on your host for Pi-hole's configuration files. This example uses `./pihole/etc-pihole` and `./pihole/etc-dnsmasq.d` relative to your `docker-compose.yml` file.
*   **Open Ports:** Ensure ports `53 (TCP/UDP)` for DNS and `80 (TCP)` for the web interface are not in use by other services on your host.
*   **Static IP for Host:** It's highly recommended that the host running Pi-hole has a static IP address on your network. You will configure your router or individual devices to use this host's IP as their primary DNS server.

## Docker Compose Script

```yaml
version: "3"

services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80/tcp" # Admin interface
    environment:
      TZ: "America/New_York" # Set your timezone, e.g., "Europe/London"
      WEBPASSWORD: "your_admin_password" # !!! IMPORTANT: CHANGE THIS PASSWORD !!!
      # Optional: Set initial upstream DNS servers. Pi-hole defaults to public DNS if not set.
      # Example: Use Cloudflare DNS
      # PIHOLE_DNS_: "1.1.1.1;1.0.0.1"
      # For specific network configurations, you might need to specify the IP
      # FTLCONF_REPLY_ADDR4: "192.168.1.100" # Replace with your host's actual LAN IP if Pi-hole fails to respond to DNS queries
    volumes:
      - "./pihole/etc-pihole:/etc/pihole"
      - "./pihole/etc-dnsmasq.d:/etc/dnsmasq.d"
    cap_add:
      - NET_ADMIN # Required for Pi-hole to function correctly
    restart: unless-stopped
```