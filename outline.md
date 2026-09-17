# Outline

Outline is a modern, collaborative knowledge base for your team or personal projects. It's designed to be simple, fast, and beautiful, allowing you to create, organize, and share documents effortlessly.

## Prerequisites

Before deploying Outline, ensure you have Docker and Docker Compose installed on your system. Outline requires a PostgreSQL database and an S3-compatible object storage solution (like MinIO, which is included in this Compose example, or an external S3 service).

## Docker Compose Script

```yaml
version: '3.8'

services:
  outline:
    image: outline/outline:latest
    container_name: outline
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      # CRITICAL: Generate a strong, random SECRET_KEY for production
      - SECRET_KEY=your_super_secret_key_here_REPLACE_ME
      - NODE_ENV=production
      # CRITICAL: Change this to your public URL (e.g., https://outline.yourdomain.com)
      - URL=http://localhost:3000
      - DATABASE_URL=postgres://outline:outline@postgres:5432/outline
      # S3-compatible storage configuration (using MinIO here)
      - S3_URL=http://minio:9000
      - S3_UPLOAD_BUCKET_URL=http://minio:9000/outline-uploads
      - S3_BUCKET_URL=http://minio:9000/outline-uploads
      - AWS_ACCESS_KEY_ID=minioadmin
      - AWS_SECRET_ACCESS_KEY=minioadmin
      - AWS_REGION=us-east-1 # MinIO doesn't strictly use regions, but Outline expects one
      - S3_FORCE_PATH_STYLE=true # Important for MinIO compatibility

    depends_on:
      - postgres
      - minio
    volumes:
      - outline_data:/var/lib/outline

  postgres:
    image: postgres:13-alpine
    container_name: outline_postgres
    restart: unless-stopped
    environment:
      - POSTGRES_DB=outline
      - POSTGRES_USER=outline
      - POSTGRES_PASSWORD=outline
    volumes:
      - postgres_data:/var/lib/postgresql/data

  minio:
    image: minio/minio:latest
    container_name: outline_minio
    restart: unless-stopped
    ports:
      - "9000:9000" # MinIO API endpoint
      - "9001:9001" # MinIO Console UI
    environment:
      - MINIO_ROOT_USER=minioadmin
      - MINIO_ROOT_PASSWORD=minioadmin
    command: server /data --console-address ":9001"
    volumes:
      - minio_data:/data

volumes:
  outline_data:
  postgres_data:
  minio_data:
```