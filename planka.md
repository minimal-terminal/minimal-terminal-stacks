# Planka: Self-Hosted Project Management

Planka is a free, open-source, and self-hosted project management tool that provides a Trello-like kanban board interface. It's designed to be simple, fast, and easy to use, making it an excellent choice for individuals or small teams looking to organize tasks, track progress, and manage projects within their homelab environment.

With Planka, you can create boards for different projects, add lists to represent stages of work (e.g., To-Do, In Progress, Done), and populate them with cards for individual tasks. Each card can have descriptions, checklists, attachments, due dates, and assigned members, providing a comprehensive overview of your project's status.

## Prerequisites

Before deploying Planka, ensure you have:
- Docker and Docker Compose installed on your host system.
- A reverse proxy (e.g., Nginx Proxy Manager, Caddy, Traefik) configured if you plan to access Planka from outside your local network.
- A dedicated directory for Planka's configuration and data.

## Docker Compose Deployment

Here's a production-ready `docker-compose.yml` for Planka, including a PostgreSQL database for persistent storage.

```yaml
version: "3.8"

services:
  planka:
    image: planka/planka:latest
    container_name: planka
    restart: unless-stopped
    depends_on:
      - planka_db
    ports:
      - "8000:80" # Change 8000 to your desired host port
    environment:
      - DATABASE_URL=postgresql://planka:planka_password@planka_db:5432/planka_db
      - TRUST_PROXY_HEADERS=1 # Set to 1 if using a reverse proxy
      - ROOT_URL=https://planka.yourdomain.com # Adjust if using a reverse proxy
    volumes:
      - ./planka_data:/data # Persistent storage for files and attachments

  planka_db:
    image: postgres:15-alpine
    container_name: planka_db
    restart: unless-stopped
    environment:
      - POSTGRES_DB=planka_db
      - POSTGRES_USER=planka
      - POSTGRES_PASSWORD=planka_password # CHANGE THIS TO A STRONG PASSWORD
    volumes:
      - ./planka_db_data:/var/lib/postgresql/data # Persistent storage for database data
```

### Setup Instructions:
1.  **Create a directory:** `mkdir planka && cd planka`
2.  **Create `docker-compose.yml`:** Paste the above content into a file named `docker-compose.yml` in your new directory.
3.  **Adjust variables:**
    *   Change `8000:80` to an available port on your host if 8000 is in use.
    *   **Crucially, change `POSTGRES_PASSWORD=planka_password` to a strong, unique password.**
    *   If using a reverse proxy, set `TRUST_PROXY_HEADERS=1` and `ROOT_URL` to your domain.
4.  **Deploy:** Run `docker compose up -d`
5.  **Access:** Open your browser to `http://your_server_ip:8000` (or your configured domain).
6.  **Initial Setup:** Follow the on-screen prompts to create your first user account.

Planka provides a robust yet simple solution for managing your tasks and projects, keeping your homelab operations organized and efficient.