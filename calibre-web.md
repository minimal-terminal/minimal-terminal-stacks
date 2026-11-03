### Calibre Web: Your Personal E-Book Server

Calibre Web is a web-based application providing a clean interface for browsing, reading, and managing your Calibre e-book library. It allows you to access your books from any device, anywhere, with robust user management and an integrated reader. Transform your home server into a powerful digital library hub.

#### Prerequisites

*   A working Docker installation.
*   An existing [Calibre](https://calibre-ebook.com/) library, or an empty directory to start one. This directory will be mounted into the container.
*   Understanding of PUID/PGID for file permissions to ensure the container can read and write to your book library.

#### `docker-compose.yml`

```yaml
version: '3.8'
services:
  calibre-web:
    image: lscr.io/linuxserver/calibre-web:latest
    container_name: calibre-web
    environment:
      - PUID=1000 # Replace with your user ID (e.g., `id -u youruser`)
      - PGID=1000 # Replace with your group ID (e.g., `id -g youruser`)
      - TZ=Etc/UTC # Replace with your local timezone, e.g., 'America/New_York'
      - DOCKER_MODS=linuxserver/calibre-web:calibre # Essential for embedded Calibre functions
      - OIDC_ENABLED=false # Set to true if integrating with OpenID Connect for authentication
    volumes:
      - /path/to/your/calibre-config:/config # Host path for Calibre Web's configuration and database
      - /path/to/your/calibre-library:/books # Host path to your existing Calibre library (e.g., `/mnt/user/books/Calibre Library`)
    ports:
      - "8083:8083" # Host_Port:Container_Port - Access Calibre Web via http://your-server-ip:8083
    restart: unless-stopped
    networks:
      - default_network # Use 'default_network' or your custom Docker network for reverse proxy integration

networks:
  default_network:
    driver: bridge
```