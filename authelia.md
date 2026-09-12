# Authelia

Authelia is an open-source authentication and authorization server providing single sign-on (SSO) and two-factor authentication (2FA) for your self-hosted applications. It acts as a companion to reverse proxies like Nginx Proxy Manager or Traefik, securing access to your services.

## Prerequisites

Before deploying Authelia, ensure you have:

*   A dedicated Docker volume for Authelia's configuration and data.
*   An external reverse proxy (e.g., Nginx Proxy Manager, Traefik) configured to forward authentication requests to Authelia.
*   A `configuration.yml` file prepared in the `./authelia/config` directory, defining your authentication backend (e.g., LDAP, YAML users), notifier (e.g., SMTP), and access policies. Refer to the official Authelia documentation for detailed configuration instructions.
*   Strong, randomly generated secrets for `AUTHELIA_JWT_SECRET` and `AUTHELIA_SESSION_SECRET`. These are critical for security and should be stored securely, ideally in an `.env` file.

## Docker Compose Script

```yaml
version: '3.8'
services:
  authelia:
    image: authelia/authelia:latest
    container_name: authelia
    restart: unless-stopped
    volumes:
      - ./authelia/config:/config # Mount your Authelia configuration directory
    environment:
      - TZ=America/New_York # Adjust to your timezone, e.g., Europe/London
      # IMPORTANT: Replace these with strong, randomly generated secrets!
      - AUTHELIA_JWT_SECRET=your_jwt_secret_replace_this_with_a_long_random_string
      - AUTHELIA_SESSION_SECRET=your_session_secret_replace_this_with_another_long_random_string
      # Optional: Configure database, LDAP, or SMTP notifier if not in configuration.yml
      # - AUTHELIA_STORAGE_POSTGRES_HOST=postgres
      # - AUTHELIA_STORAGE_POSTGRES_DATABASE=authelia
      # - AUTHELIA_STORAGE_POSTGRES_USERNAME=authelia
      # - AUTHELIA_STORAGE_POSTGRES_PASSWORD=your_postgres_password
      # - AUTHELIA_AUTHENTICATION_BACKEND_LDAP_URL=ldap://your-ldap-server:389
      # - AUTHELIA_AUTHENTICATION_BACKEND_LDAP_BASE_DN=dc=example,dc=com
      # - AUTHELIA_AUTHENTICATION_BACKEND_LDAP_USERNAME=cn=admin,dc=example,dc=com
      # - AUTHELIA_AUTHENTICATION_BACKEND_LDAP_PASSWORD=your_ldap_admin_password
      # - AUTHELIA_NOTIFIER_SMTP_HOST=smtp.yourdomain.com
      # - AUTHELIA_NOTIFIER_SMTP_PORT=587
      # - AUTHELIA_NOTIFIER_SMTP_USERNAME=your-email@yourdomain.com
      # - AUTHELIA_NOTIFIER_SMTP_PASSWORD=your_smtp_password
      # - AUTHELIA_NOTIFIER_SMTP_SENDER=Authelia <your-email@yourdomain.com>
    ports:
      - "9091:9091" # Authelia's default HTTP port. This port should typically NOT be exposed directly to the internet.
                  # It's intended for your reverse proxy to connect to.
    # network_mode: bridge # Default, or custom network if needed for your reverse proxy
    # You will need to configure your reverse proxy (Nginx Proxy Manager, Traefik, Caddy)
    # to use Authelia's /api/verify endpoint for forward authentication.
```