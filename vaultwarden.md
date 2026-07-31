# Vaultwarden (Bitwarden-compatible server)

Vaultwarden, formerly known as Bitwarden_RS, is an unofficial, self-hosted Bitwarden server implementation written in Rust. It is compatible with the official Bitwarden clients and provides a lightweight, secure, and efficient solution for managing your passwords and sensitive information on your own infrastructure. It's an excellent choice for those who want the full functionality of Bitwarden without running the resource-intensive official server.

## Prerequisites

Ensure you have a Docker environment set up. A dedicated volume for Vaultwarden's data (`/data`) is highly recommended to persist all your password vault information. Port `80` (or another chosen port) will be exposed for web access.

## Docker Compose Script

```yaml
version: '3.8'

services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    ports:
      - "80:80" # Standard HTTP port for web access
      # - "3012:3012" # Optional: WebSocket port for push notifications (uncomment if needed)
    volumes:
      - ./data:/data # Persist data to a local directory
    environment:
      # - WEBSOCKET_ENABLED=true # Uncomment if using WebSocket port
      - SIGNUPS_ALLOWED=false # Set to true initially if you need to create an admin user, then change to false for security
      - ADMIN_TOKEN=your_strong_admin_token # REQUIRED: Set a strong token for admin access (e.g., /admin)
      - DOMAIN=https://your.domain.com # Optional: Set your domain for proper certificate handling and links
      # - LOG_LEVEL=warn # Optional: Adjust log level (trace, debug, info, warn, error)
      # - ROCKET_ADDRESS=0.0.0.0 # Optional: Listen on all interfaces (default)
      # - ROCKET_PORT=80 # Optional: Port rocket will listen on (default)
      # - SMTP_HOST=smtp.your-email.com # Optional: For email notifications (e.g., password reset)
      # - SMTP_FROM=vaultwarden@your-email.com
      # - SMTP_PORT=587
      # - SMTP_SSL=true
      # - SMTP_USERNAME=your-email-username
      # - SMTP_PASSWORD=your-email-password
      # - SMTP_AUTH_MECHANISM=Login
```