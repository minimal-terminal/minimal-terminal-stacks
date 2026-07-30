# BookStack

BookStack is a simple, self-hosted, easy-to-use platform for organizing and storing information. It's perfect for personal notes, team wikis, or extensive project documentation, offering a structured way to manage knowledge with books, chapters, and pages.

## Prerequisites

Before deploying BookStack, ensure you have a Docker environment set up. You'll need to create a Docker volume for persistent data storage and choose an available port on your host system for accessing the web interface.

## Docker Compose Script

```yaml
version: '3.8'

services:
  bookstack:
    image: lscr.io/linuxserver/bookstack:latest
    container_name: bookstack
    environment:
      - PUID=1000
      - PGID=1000
      - APP_URL=http://your-domain.com # Change to your BookStack URL or leave blank for auto-detection
      - DB_HOST=bookstack_db
      - DB_DATABASE=bookstack
      - DB_USERNAME=bookstackuser
      - DB_PASSWORD=your_db_password # <--- CHANGE THIS SECURE PASSWORD
    volumes:
      - ./config:/config
    ports:
      - 6875:80 # Host_Port:Container_Port - Change 6875 to an unused port on your host
    restart: unless-stopped
    depends_on:
      - bookstack_db

  bookstack_db:
    image: mariadb:10.5 # Use a specific version for stability
    container_name: bookstack_db
    environment:
      - PUID=1000
      - PGID=1000
      - MYSQL_ROOT_PASSWORD=your_root_password # <--- CHANGE THIS SECURE PASSWORD
      - MYSQL_DATABASE=bookstack
      - MYSQL_USER=bookstackuser
      - MYSQL_PASSWORD=your_db_password # <--- MUST MATCH DB_PASSWORD ABOVE
    volumes:
      - ./database:/config
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
```