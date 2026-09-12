## WireGuard

WireGuard is a modern, fast, and secure VPN protocol that utilizes state-of-the-art cryptography. This Docker Compose script deploys a WireGuard server, allowing you to create encrypted tunnels to your home network for secure remote access.

## Prerequisites

*   Docker and Docker Compose installed on your system.
*   A dedicated directory for WireGuard's configuration files, e.g., `./wireguard/config`.
*   Port forwarding configured on your router for the chosen UDP port (default 51820) to your Docker host's IP address.
*   The `NET_ADMIN` and `SYS_MODULE` capabilities are essential for WireGuard to operate correctly within a Docker container.

## Docker Compose Script

```yaml
version: "3.8"
services:
  wireguard:
    image: linuxserver/wireguard:latest
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC # Set your timezone, e.g., America/New_York
      - SERVERURL=your_domain.com # Replace with your public IP or domain name
      - SERVERPORT=51820 # Adjust if you want a different port
      - PEERS=1 # Number of client peers to generate initially
      - PEERDNS=auto # Or specify DNS servers, e.g., 1.1.1.1,8.8.8.8
      - INTERNAL_SUBNET=10.13.13.0/24 # Internal VPN network subnet
    volumes:
      - ./wireguard/config:/config
      - /lib/modules:/lib/modules # Required for kernel modules
    ports:
      - 51820:51820/udp # UDP port for WireGuard
    sysctls:
      - net.ipv4.conf.all.src_valid_lables=1
      - net.ipv4.ip_forward=1
    restart: unless-stopped
```