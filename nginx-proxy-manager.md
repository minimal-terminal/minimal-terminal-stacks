# Nginx Proxy Manager

Nginx Proxy Manager is a powerful and easy-to-use reverse proxy management system built on Nginx, featuring a beautiful web interface. It allows you to expose your self-hosted applications to the internet securely and easily, automatically managing SSL certificates from Let's Encrypt.

## Prerequisites

Before deploying Nginx Proxy Manager, ensure you have Docker and Docker Compose installed. You'll need to map ports 80 and 443 from your host to the container for HTTP/HTTPS traffic, and port 81 for the administrative web interface. Persistent volumes are used to store configuration data and SSL certificates.

## Docker Compose Script

```yaml
version: '3'

services:
  nginx-proxy-manager:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: nginx-proxy-manager
    restart: unless-stopped
    ports:
      - '80:80'    # Public HTTP port
      - '443:443'  # Public HTTPS port
      - '81:81'    # Admin UI port
    volumes:
      - npm_data:/data
      - npm_letsencrypt:/etc/letsencrypt
    # Optional: Set user/group ID if you have specific permissions needs
    # environment:
    #   - PUID=1000
    #   - PGID=1000

volumes:
  npm_data:
  npm_letsencrypt:
```