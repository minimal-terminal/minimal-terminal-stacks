```yaml
version: "3.8"

services:
  plausible_db:
    image: postgres:14-alpine
    restart: unless-stopped
    volumes:
      - plausible_db_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: plausible
      POSTGRES_USER: plausible
      POSTGRES_PASSWORD: ${PLAUSIBLE_DB_PASSWORD}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - plausible_network

  plausible_events_db:
    image: clickhouse/clickhouse-server:23.8.3.48-alpine
    restart: unless-stopped
    volumes:
      - plausible_events_db_data:/var/lib/clickhouse
    environment:
      CLICKHOUSE_USER: plausible
      CLICKHOUSE_PASSWORD: ${PLAUSIBLE_CLICKHOUSE_PASSWORD}
      CLICKHOUSE_DB: plausible
    ulimits:
      nofile:
        soft: 262144
        hard: 262144
    healthcheck:
      test: ["CMD", "curl", "--fail", "http://localhost:8123"]
      interval: 30s
      timeout: 10s
      retries: 3
    networks:
      - plausible_network

  plausible:
    image: plausible/analytics:v1.30.0
    restart: unless-stopped
    command: sh -c "sleep 10 && /entrypoint.sh db createdb && /entrypoint.sh db migrate && /entrypoint.sh run"
    depends_on:
      plausible_db:
        condition: service_healthy
      plausible_events_db:
        condition: service_healthy
    ports:
      - "8000:8000" # Map to host port 8000, or use a reverse proxy like Nginx Proxy Manager/Caddy
    environment:
      DATABASE_URL: postgresql://plausible:${PLAUSIBLE_DB_PASSWORD}@plausible_db:5432/plausible
      CLICKHOUSE_DATABASE_URL: http://plausible:${PLAUSIBLE_CLICKHOUSE_PASSWORD}@plausible_events_db:8123/plausible
      BASE_URL: ${PLAUSIBLE_BASE_URL} # e.g., https://analytics.example.com
      SECRET_KEY_BASE: ${PLAUSIBLE_SECRET_KEY_BASE} # Generate with 'openssl rand -base64 64'
      # Optional: Uncomment and configure for admin user creation and email
      # ADMIN_USER_EMAIL: admin@example.com
      # ADMIN_USER_NAME: Admin
      # ADMIN_USER_PASSWORD: ${PLAUSIBLE_ADMIN_PASSWORD}
      # DISABLE_REGISTRATION: "true" # Uncomment to disable public registration
    networks:
      - plausible_network

volumes:
  plausible_db_data:
  plausible_events_db_data:

networks:
  plausible_network:
    driver: bridge
```