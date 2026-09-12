# Gitea

Gitea is a painless self-hosted Git service. It's a lightweight, open-source alternative to GitHub or GitLab, perfect for personal projects, small teams, or managing configuration files on a home server.

## Prerequisites

Before deploying Gitea, ensure you have:

*   A Docker volume for persistent Gitea configuration and database (e.g., `gitea_data`).
*   A Docker volume for the Git repositories themselves (e.g., `gitea_git`).
*   Ensure ports `3000` (HTTP/HTTPS web interface) and `2222` (mapped to container's SSH port `22`) are available on your host.

## Docker Compose Script

```yaml
version: '3'

services:
  gitea:
    image: gitea/gitea:latest
    container_name: gitea
    environment:
      - USER_UID=1000 # Change to your user's UID on the host for correct file permissions
      - USER_GID=1000 # Change to your user's GID on the host for correct file permissions
      - GITEA__SERVER__ROOT_URL=http://gitea.your-domain.com/ # Adjust if using a reverse proxy, or use http://your-nas-ip:3000/
      - GITEA__SERVER__SSH_DOMAIN=gitea.your-domain.com # Adjust if using a reverse proxy, or use your-nas-ip
      - GITEA__SERVER__SSH_PORT=2222 # The host port mapped to container's SSH port 22
      - GITEA__DATABASE__TYPE=sqlite3
      - GITEA__DATABASE__PATH=/data/gitea.db
      # For PostgreSQL/MySQL, uncomment and configure these (and add a 'db' service):
      # - GITEA__DATABASE__TYPE=postgres
      # - GITEA__DATABASE__HOST=db:5432
      # - GITEA__DATABASE__NAME=gitea
      # - GITEA__DATABASE__USER=gitea
      # - GITEA__DATABASE__PASS=gitea_password
    volumes:
      - gitea_data:/data
      - gitea_git:/git
      # If you want to use a specific host path for git repositories directly, uncomment and modify:
      # - /path/to/your/host/git/repos:/git
    ports:
      - "3000:3000" # Gitea HTTP/HTTPS web interface
      - "2222:22"   # Gitea SSH port (host:container). Mapped to 2222 on host to avoid conflict with host SSH.
    restart: unless-stopped
    # If using an external database (e.g., PostgreSQL or MySQL), uncomment depends_on and define the 'db' service:
    # depends_on:
    #   - db # Name of your database service

volumes:
  gitea_data:
  gitea_git:
```