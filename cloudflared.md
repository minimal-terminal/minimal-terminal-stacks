# Cloudflared (Cloudflare Tunnel)

Cloudflared is the daemon that powers Cloudflare Tunnel, allowing you to securely connect your self-hosted services to Cloudflare's global network without opening any inbound ports on your firewall. It establishes an outbound-only connection to Cloudflare, providing a secure, zero-trust way to expose your applications.

## Prerequisites

1.  **Cloudflare Account:** You need an active Cloudflare account.
2.  **Domain:** A domain managed by Cloudflare is required.
3.  **Cloudflare Tunnel:** Configure a tunnel via the Cloudflare Zero Trust dashboard. You will need to obtain your tunnel token (e.g., `eyJhIjoi...`) from the Cloudflare dashboard when setting up your tunnel. This token is crucial for the `cloudflared` daemon to authenticate and establish the tunnel.

## Docker Compose Script

```yaml
version: '3.8'

services:
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    restart: unless-stopped
    command: tunnel --no-autoupdate run --token YOUR_TUNNEL_TOKEN
    environment:
      - TZ=America/New_York # Set your desired timezone
    # Use 'host' network mode to allow cloudflared to directly access
    # services running on the host machine or other Docker containers
    # publishing ports to the host's network interfaces.
    # For more complex setups where other Docker containers are on a
    # specific Docker bridge network, you might want cloudflared to join
    # that network instead (e.g., using 'networks: - my_internal_network').
    network_mode: host
    # Volumes are not strictly necessary for simple token-based tunnels,
    # but can be used for persistent configuration files if needed.
    # For example, to use a config.yml:
    # volumes:
    #   - ./cloudflared:/etc/cloudflared
    # And modify the command to 'tunnel --config /etc/cloudflared/config.yml run'
```

**Important:** Replace `YOUR_TUNNEL_TOKEN` with the actual token obtained from your Cloudflare Zero Trust dashboard. For more advanced configurations involving multiple services or specific routing rules, consider using a `config.yml` mounted as a volume.