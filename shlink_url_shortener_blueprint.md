```yaml
version: '3.8'
services:
  shlink-db:
    image: mariadb:10.6
    container_name: shlink-db
    restart: unless-stopped
    environment:
      MARIADB_ROOT_PASSWORD: your_root_password # CRITICAL: Change this strong password!
      MARIADB_DATABASE: shlink_db
      MARIADB_USER: shlink_user
      MARIADB_PASSWORD: your_shlink_password # CRITICAL: Change this strong password!
    volumes:
      - shlink_db_data:/var/lib/mysql
    networks:
      - shlink_network

  shlink:
    image: shlinkio/shlink:stable
    container_name: shlink
    restart: unless-stopped
    ports:
      - "8080:8080" # Map to a free port on your host, or expose via reverse proxy (recommended)
    environment:
      # Database connection
      DB_CONNECTION: 'mysql'
      DB_HOST: 'shlink-db'
      DB_PORT: '3306'
      DB_NAME: 'shlink_db'
      DB_USER: 'shlink_user'
      DB_PASSWORD: 'your_shlink_password' # Must match shlink-db service password
      # Shlink configuration (adjust as needed)
      DEFAULT_DOMAIN: 'your-short-domain.com' # Replace with your actual short domain
      IS_HTTPS_ENABLED: 'true' # Set to true if using HTTPS (recommended via reverse proxy)
      GEOLITE_LICENSE_KEY: 'your_geolite_license_key' # Optional: Register at MaxMind for geo-location stats
    volumes:
      - shlink_data:/etc/shlink
    depends_on:
      - shlink-db
    networks:
      - shlink_network

volumes:
  shlink_db_data:
  shlink_data:

networks:
  shlink_network:
    driver: bridge
```