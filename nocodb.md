# NocoDB

NocoDB is an open-source low-code platform that turns any database into a smart spreadsheet. It provides a rich spreadsheet interface for your data, allowing you to build custom applications, workflows, and dashboards with ease, similar to Airtable. It supports various databases like MySQL, PostgreSQL, SQLite, and more.

## Prerequisites

Before deploying NocoDB, ensure you have Docker and Docker Compose installed on your system.
You will need to create a persistent volume for NocoDB's configuration and data. A dedicated folder on your host machine, for example, `/path/to/your/nocodb_data`, will be mapped into the container.
NocoDB typically runs on port `8080`. Ensure this port is not already in use on your host machine or adjust it as needed.

## Docker Compose Script

```yaml
version: '3.8'

services:
  nocodb:
    image: nocodb/nocodb:latest
    container_name: nocodb
    ports:
      - "8080:8080"
    volumes:
      - /path/to/your/nocodb_data:/usr/app/data
    environment:
      # Optional: Set the database type for NocoDB's internal metadata.
      # SQLite is used by default if no external DB is configured for internal metadata.
      # If you want to use an external database for NocoDB's internal metadata,
      # uncomment and configure the following variables.
      # For example, using PostgreSQL:
      # NC_DB: pg
      # NC_DB_HOST: your_postgres_host
      # NC_DB_PORT: 5432
      # NC_DB_USER: your_postgres_user
      # NC_DB_PASSWORD: your_postgres_password
      # NC_DB_NAME: nocodb_meta
      
      # Optional: Set the application port if different from 8080
      # PORT: 8080
      
      # Optional: Set the base URL if NocoDB is behind a reverse proxy
      # NC_PUBLIC_URL: https://nocodb.yourdomain.com
      
      # Optional: JWT Secret for API authentication
      # NC_JWT_SECRET: your_secure_jwt_secret
    restart: unless-stopped
```