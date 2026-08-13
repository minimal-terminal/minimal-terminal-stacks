# Baserow

Baserow is an open-source no-code database and spreadsheet tool that allows users to build custom applications and manage data with the flexibility of a spreadsheet and the power of a relational database. It's a robust alternative to tools like Airtable, offering powerful API access and real-time collaboration features.

## Prerequisites

Before deploying Baserow, ensure you have Docker and Docker Compose installed on your system.
You will need to create a `.env` file in the same directory as your `docker-compose.yml` with a strong password for the database:

```
DB_PASSWORD=your_strong_database_password_here
```

Ensure you replace `your_strong_database_password_here` with a secure, randomly generated password.
Also, remember to adjust `BASEROW_PUBLIC_URL` and `BASEROW_CORS_ORIGIN` in the `docker-compose.yml` to your actual domain or IP address where Baserow will be accessible (e.g., `http://your_domain.com:8000` or `http://192.168.1.100:8000`).

## Docker Compose Script

```yaml
version: "3.9"

services:
  baserow:
    image: baserow/baserow:1.24.2 # Use the latest stable version
    container_name: baserow
    restart: unless-stopped
    environment:
      BASEROW_PUBLIC_URL: "http://your_domain_or_ip:8000" # IMPORTANT: Change this to your actual Baserow URL
      DATABASE_HOST: db
      DATABASE_NAME: baserow
      DATABASE_USER: baserow
      DATABASE_PASSWORD: ${DB_PASSWORD}
      BASEROW_CORS_ORIGIN: "http://your_domain_or_ip:8000" # IMPORTANT: Change this to your actual Baserow URL
    volumes:
      - baserow_data:/baserow/data
    ports:
      - "8000:80" # Host port 8000 -> Container port 80 (Baserow web interface)
    depends_on:
      - db

  db:
    image: postgres:15-alpine
    container_name: baserow_db
    restart: unless-stopped
    environment:
      POSTGRES_DB: baserow
      POSTGRES_USER: baserow
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - baserow_pg_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  baserow_data:
  baserow_pg_data:
```