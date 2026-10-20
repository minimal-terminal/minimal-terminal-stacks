```yaml
version: '3.8'

services:
  db:
    image: mariadb:10.6
    container_name: monica_db
    environment:
      MYSQL_ROOT_PASSWORD: "your_root_password" # CHANGE THIS to a strong password
      MYSQL_DATABASE: monica
      MYSQL_USER: monica
      MYSQL_PASSWORD: "your_monica_db_password" # CHANGE THIS to a strong password
    volumes:
      - ./monica_db:/var/lib/mysql
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      timeout: 20s
      retries: 10
      start_period: 30s

  monica:
    image: monica:latest
    container_name: monica_app
    environment:
      APP_ENV: production
      APP_KEY: "base64:your_app_key_here" # IMPORTANT: Generate with `docker exec -it monica_app php artisan key:generate --show` after initial startup
      DB_CONNECTION: mysql
      DB_HOST: db
      DB_PORT: 3306
      DB_DATABASE: monica
      DB_USERNAME: monica
      DB_PASSWORD: "your_monica_db_password" # Must match the DB service password
      APP_URL: "https://monica.yourdomain.com" # CHANGE THIS to your actual domain
      # Optional: Configure Mail for notifications (uncomment and fill in details)
      # MAIL_MAILER: smtp
      # MAIL_HOST: your_mail_host
      # MAIL_PORT: 587
      # MAIL_USERNAME: your_mail_username
      # MAIL_PASSWORD: your_mail_password
      # MAIL_ENCRYPTION: tls
      # MAIL_FROM_ADDRESS: no-reply@yourdomain.com
      # MAIL_FROM_NAME: Monica
    volumes:
      - ./monica_data:/var/www/html/storage
    ports:
      - "8000:80" # Map to an available port on your host, or remove if using a reverse proxy
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped
```