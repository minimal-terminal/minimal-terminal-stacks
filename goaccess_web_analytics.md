# GoAccess Self-Hosted Web Analytics Blueprint

This blueprint guides you through deploying a self-hosted web analytics dashboard using GoAccess, a real-time web log analyzer and interactive viewer that runs in a terminal or through your browser. Reclaim your privacy and gain deep insights into your web traffic without relying on external services.

## 📋 Prerequisites

Before you begin, ensure you have the following:

*   **A Web Server:** Nginx or Apache with active access logs. This guide assumes Nginx.
*   **Docker & Docker Compose:** Installed on your server.
*   **Basic Linux CLI Knowledge:** Familiarity with `cd`, `mkdir`, `nano` or `vim`.
*   **Log File Access:** GoAccess will need read access to your web server's access logs.

## 🚀 Step 1: Prepare Your Web Server Logs (Nginx Example)

GoAccess works best with a consistent log format. Ensure your Nginx configuration includes a `log_format` directive that GoAccess can easily parse. A common format is `combined`.

Edit your Nginx configuration (e.g., `/etc/nginx/nginx.conf` or a site-specific config in `/etc/nginx/sites-available/your_site.conf`):

```nginx
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log main;
    error_log   /var/log/nginx/error.log warn;
    # ... other http settings
}

server {
    # ... your server block
    access_log /var/log/nginx/your_site_access.log main; # Ensure this is enabled and uses the 'main' format
    # ...
}
```

After modifying, test your Nginx configuration and reload it:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

Note the path to your access log file (e.g., `/var/log/nginx/your_site_access.log`). You'll need this for Docker Compose.

## 🐳 Step 2: Create Docker Compose Configuration

Create a new directory for your GoAccess project and navigate into it:

```bash
mkdir ~/goaccess-analytics
cd ~/goaccess-analytics
```

Create a `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  goaccess:
    image: allinurl/goaccess:latest
    container_name: goaccess
    restart: unless-stopped
    ports:
      - "7888:7888" # Port to access the real-time dashboard
    volumes:
      - /var/log/nginx:/var/log/nginx:ro # Mount your Nginx log directory
      - ./goaccess.conf:/etc/goaccess/goaccess.conf:ro # Optional: Custom GoAccess config
    command: >
      --log-file /var/log/nginx/your_site_access.log # IMPORTANT: Update with your log file path
      --output-format html
      --real-time-html
      --port 7888
      --ws-url ws://your.domain.com:7888/ws # IMPORTANT: Update with your actual domain/IP or 'ws://YOUR_SERVER_IP:7888/ws'
      --config-file /etc/goaccess/goaccess.conf

```

**IMPORTANT:**

*   Replace `/var/log/nginx/your_site_access.log` with the actual path to your Nginx (or Apache) access log file.
*   Replace `your.domain.com` with your server's IP address or domain name where GoAccess will be accessible. If you're accessing it directly via IP, use `ws://YOUR_SERVER_IP:7888/ws`.
*   The `volumes` section mounts your host's Nginx log directory into the container as read-only (`:ro`). This is crucial for GoAccess to access the logs.

## ⚙️ Step 3: (Optional) Custom GoAccess Configuration

If you want to customize GoAccess further (e.g., change date/time formats, ignore specific user agents), create a `goaccess.conf` file in the same directory as your `docker-compose.yml`.

You can start by copying the default configuration from the container:

```bash
docker run --rm allinurl/goaccess cat /etc/goaccess/goaccess.conf > goaccess.conf
```

Then, edit `goaccess.conf` as needed. For Nginx `combined` log format, ensure these lines are correctly set (they usually are by default, but double-check):

```ini
# Log Format
log-format COMBINED

# Date Format
date-format %d/%b/%Y

# Time Format
time-format %H:%M:%S
```

If your `ws-url` in `docker-compose.yml` uses a domain behind a reverse proxy (e.g., Caddy, Nginx Proxy Manager), you might need to adjust the `ws-url` to reflect the external domain and port, and ensure your reverse proxy correctly forwards WebSocket traffic.

## ▶️ Step 4: Deploy GoAccess

From within your `~/goaccess-analytics` directory, deploy the container:

```bash
docker compose up -d
```

GoAccess will start parsing your logs in real-time.

## 📊 Step 5: Access Your Dashboard

Open your web browser and navigate to `http://YOUR_SERVER_IP:7888` (replace `YOUR_SERVER_IP` with the actual IP address or domain of your server).

You should now see an interactive, real-time web analytics dashboard powered by GoAccess! The data will update automatically as new requests hit your web server.

## 🧹 Cleanup and Maintenance

To stop and remove the GoAccess container:

```bash
docker compose down
```

To view logs from the GoAccess container:

```bash
docker compose logs -f goaccess
```

This setup provides a robust, privacy-focused way to monitor your web server traffic. Enjoy your new insights!
