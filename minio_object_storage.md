# Deploy Self-Hosted S3 Object Storage with MinIO

Transform your homelab into a powerful object storage platform this weekend. MinIO provides an S3-compatible API, making it an excellent choice for secure backups, media archives, and application storage. This blueprint will guide you through setting up a robust, scalable object storage solution that you fully control.

## Prerequisites

Before you begin, ensure you have the following:

*   A Linux server (e.g., Raspberry Pi, VM, or bare metal)
*   Docker and Docker Compose installed
*   Basic terminal proficiency

## Step-by-Step Guide

### Step 1: Prepare Your Environment

First, create a dedicated directory for your MinIO configuration and data. This ensures a clean and organized setup.

```bash
mkdir -p ~/docker/minio
cd ~/docker/minio
```

### Step 2: Create a `docker-compose.yml`

Next, create a `docker-compose.yml` file within your `~/docker/minio` directory. This file defines the MinIO service, its ports, persistent volumes, and environment variables.

```yaml
version: "3.8"

services:
  minio:
    image: minio/minio:RELEASE.2024-06-16T12-32-00Z # Use a specific, stable release tag
    container_name: minio
    ports:
      - "9000:9000" # MinIO S3 API port
      - "9001:9001" # MinIO Console port
    volumes:
      - ./data:/data # Persistent storage for MinIO data
      - ./config:/root/.minio # Configuration files
    environment:
      MINIO_ROOT_USER: ${MINIO_ROOT_USER}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
      # MINIO_SERVER_URL: "http://your-server-ip:9000" # Uncomment and set if behind a reverse proxy or for console access
    command: server /data --console-address ":9001"
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
```

### Step 3: Define Credentials in `.env`

For security, define your MinIO root user and password in a `.env` file in the same directory (`~/docker/minio`).

```ini
# MinIO Credentials
MINIO_ROOT_USER=your_minio_admin_username
MINIO_ROOT_PASSWORD=your_strong_minio_password
```

**CRITICAL:** Replace `your_minio_admin_username` and `your_strong_minio_password` with secure, unique values. Do not use default or weak credentials.

### Step 4: Start MinIO

With your `docker-compose.yml` and `.env` files ready, deploy MinIO using Docker Compose.

```bash
docker compose up -d
```

This command will pull the MinIO image (if not already present), create the container, and start it in detached mode.

### Step 5: Access MinIO

Once the container is running, you can access your MinIO instance:

*   **MinIO Console:** Open your web browser and navigate to `http://your-server-ip:9001`.
*   Log in using the `MINIO_ROOT_USER` and `MINIO_ROOT_PASSWORD` you defined in your `.env` file.

From the console, you can create buckets, upload objects, manage users, and configure your S3-compatible storage. You can also connect using any S3-compatible client (e.g., `mc` CLI, Cyberduck, S3FS) to `http://your-server-ip:9000`.

### Step 6: (Optional) Secure with TLS/Reverse Proxy

For production use, or if you plan to access MinIO over the internet, it is highly recommended to place it behind a reverse proxy (like Caddy or Nginx Proxy Manager) to enable HTTPS. This encrypts your data in transit.

If using a reverse proxy, you might uncomment and update `MINIO_SERVER_URL` in your `.env` file to reflect your HTTPS domain (e.g., `https://minio.yourdomain.com`).

### Step 7: (Optional) Systemd Service for Docker Compose

To ensure your MinIO instance starts automatically on boot and can be managed easily with `systemctl`:

1.  Create a systemd service file: `/etc/systemd/system/minio-docker.service`

    ```ini
    [Unit]
    Description=MinIO Docker Compose Service
    Requires=docker.service
    After=docker.service

    [Service]
    WorkingDirectory=/home/youruser/docker/minio # Adjust this path to your minio directory
    ExecStart=/usr/local/bin/docker compose up -d --remove-orphans
    ExecStop=/usr/local/bin/docker compose down
    Type=oneshot
    RemainAfterExit=yes
    User=youruser # Replace with your non-root user that owns the docker directory
    Group=docker

    [Install]
    WantedBy=multi-user.target
    ```

2.  Reload the systemd daemon and enable/start the service:

    ```bash
sudo systemctl daemon-reload
sudo systemctl enable minio-docker.service
sudo systemctl start minio-docker.service
    ```

    **Important:** Replace `youruser` with the actual non-root user that owns the `~/docker/minio` directory.

## Conclusion

Congratulations! You now have a powerful, self-hosted S3-compatible object storage system running in your homelab. Explore its capabilities with S3 clients, integrate it into your backup routines, or use it as the backend for your applications. The possibilities are vast when you control your own data.