version: '3.8'

services:
  db:
    image: postgres:15-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: miniflux_user
      POSTGRES_PASSWORD: your_strong_db_password # <--- CHANGE THIS
      POSTGRES_DB: miniflux_db
    volumes:
      - miniflux_db_data:/var/lib/postgresql/data
    networks:
      - miniflux_net

  miniflux:
    image: miniflux/miniflux:latest
    restart: unless-stopped
    ports:
      - "8000:8080" # Map host port 8000 to container port 8080. Adjust if 8000 is in use.
    environment:
      DATABASE_URL: postgres://miniflux_user:your_strong_db_password@db:5432/miniflux_db?sslmode=disable
      BASE_URL: http://your_server_ip:8000 # <--- CHANGE THIS to your actual URL (e.g., https://rss.yourdomain.com)
      RUN_MIGRATIONS: "1"
      CREATE_ADMIN: "1"
      ADMIN_USERNAME: admin # <--- CHANGE THIS
      ADMIN_PASSWORD: your_strong_admin_password # <--- CHANGE THIS
      TZ: America/New_York # <--- Adjust to your timezone
    depends_on:
      - db
    networks:
      - miniflux_net

volumes:
  miniflux_db_data:

networks:
  miniflux_net:
    driver: bridge