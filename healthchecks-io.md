# Healthchecks.io: Robust Monitoring for Your Scheduled Tasks

Healthchecks.io is an open-source "dead man's switch" that monitors your cron jobs, background services, and scheduled scripts. Instead of checking *if* a service is down, it checks *if* a service is still *up* by expecting periodic pings. If a ping is missed, it alerts you, ensuring your critical automated tasks never silently fail.

## Prerequisites
- Docker and Docker Compose installed.
- A reverse proxy (like Caddy, Nginx Proxy Manager, or Traefik) is highly recommended for HTTPS access and custom domains. If using a reverse proxy, ensure the `ports` mapping for the `web` service is removed or configured not to conflict.

## docker-compose.yml
```yaml
version: '3.8'

services:
  db:
    image: postgres:15-alpine
    container_name: healthchecks_db
    restart: unless-stopped
    volumes:
      - ./data/healthchecks_db:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: healthchecks
      POSTGRES_USER: healthchecks
      POSTGRES_PASSWORD: your_strong_db_password_here # << CHANGE THIS to a strong, unique password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U healthchecks -d healthchecks"]
      interval: 10s
      timeout: 5s
      retries: 5

  web:
    image: ghcr.io/healthchecks/healthchecks:latest
    container_name: healthchecks_web
    restart: unless-stopped
    volumes:
      - ./data/healthchecks_media:/opt/healthchecks/media
      - ./data/healthchecks_static:/opt/healthchecks/static
    environment:
      DATABASE_URL: postgres://healthchecks:your_strong_db_password_here@db/healthchecks # << Must match db service password
      SECRET_KEY: your_super_secret_django_key_here # << CHANGE THIS to a strong, random Django secret key
      ALLOWED_HOSTS: your.domain.com,localhost,127.0.0.1 # << CHANGE THIS to your domain, comma-separated
      SITE_ROOT: https://your.domain.com # << CHANGE THIS to your full URL
      
      # --- Email Settings for Alerts (Highly Recommended) ---
      EMAIL_HOST: smtp.your_email_provider.com # e.g., smtp.sendgrid.net, smtp.mailgun.org
      EMAIL_PORT: 587 # e.g., 587 (TLS) or 465 (SSL)
      EMAIL_HOST_USER: your_smtp_username # << CHANGE THIS
      EMAIL_HOST_PASSWORD: your_smtp_password # << CHANGE THIS
      EMAIL_USE_TLS: 'True' # Use 'False' if your SMTP uses SSL on port 465
      DEFAULT_FROM_EMAIL: healthchecks@your.domain.com # << CHANGE THIS
      
      # --- Optional: Telegram Bot for Alerts ---
      # TELEGRAM_TOKEN: your_telegram_bot_token # Uncomment and set if using Telegram
      # TELEGRAM_CHANNEL: your_telegram_chat_id # Uncomment and set if using Telegram
      
    ports:
      - "8000:8000" # Exposes the web interface. REMOVE or adjust if using a reverse proxy.
    depends_on:
      db:
        condition: service_healthy
    command: ["gunicorn", "--bind", "0.0.0.0:8000", "hc.wsgi:application"]

  worker:
    image: ghcr.io/healthchecks/healthchecks:latest
    container_name: healthchecks_worker
    restart: unless-stopped
    environment:
      DATABASE_URL: postgres://healthchecks:your_strong_db_password_here@db/healthchecks # << Must match db service password
      SECRET_KEY: your_super_secret_django_key_here # << Must match web service secret key
      # Duplicate email/telegram settings from web service, as the worker sends alerts
      EMAIL_HOST: smtp.your_email_provider.com
      EMAIL_PORT: 587
      EMAIL_HOST_USER: your_smtp_username
      EMAIL_HOST_PASSWORD: your_smtp_password
      EMAIL_USE_TLS: 'True'
      DEFAULT_FROM_EMAIL: healthchecks@your.domain.com
      # TELEGRAM_TOKEN: your_telegram_bot_token
      # TELEGRAM_CHANNEL: your_telegram_chat_id
    depends_on:
      db:
        condition: service_healthy
    command: ["python", "manage.py", "run_q"] # This command processes the alert queue.
```

## Deployment Instructions
1.  **Save the file**: Save the content above as `docker-compose.yml` in a new directory (e.g., `~/healthchecks`).
2.  **Create data directories**: `mkdir -p ./data/healthchecks_db ./data/healthchecks_media ./data/healthchecks_static`
3.  **Configure Environment Variables**: Replace all placeholder values (e.g., `your_strong_db_password_here`, `your_super_secret_django_key_here`, `your.domain.com`, and all `EMAIL_` variables) with your actual, secure credentials and domain.
4.  **Initial Database Migration**: Before starting the services, run the database migrations:
    ```bash
    docker compose run --rm web python manage.py migrate
    ```
5.  **Create Superuser (Optional but Recommended)**: Create an admin user for the web interface:
    ```bash
    docker compose run --rm web python manage.py createsuperuser
    ```
6.  **Start the Stack**: Navigate to your directory and run:
    ```bash
    docker compose up -d
    ```
7.  **Access Healthchecks.io**: If you've mapped port 8000, you can access it at `http://localhost:8000` or `http://your_server_ip:8000`. For production, configure your reverse proxy to route traffic from `https://your.domain.com` to the `web` service on port `8000` (e.g., `http://healthchecks_web:8000` within your Docker network).

Healthchecks.io will now be running, ready to receive pings and alert you when your critical tasks go silent.