# Deploy a Privacy-Focused Code Snippet & Pastebin with PrivateBin

This blueprint guides you through deploying PrivateBin, a minimalist, open-source online pastebin where the server has zero knowledge of pasted data. Data is encrypted/decrypted in the browser using 256-bit AES. It's perfect for securely sharing code snippets, configuration files, or any sensitive text without worrying about server-side snooping.

## Prerequisites

*   A server (physical or virtual) running Linux.
*   Docker and Docker Compose installed.
*   (Optional but Recommended) A reverse proxy like Nginx Proxy Manager or Caddy for secure external access with SSL. For this guide, we'll expose it on a host port, assuming you'll handle reverse proxying separately if needed.

## Step-by-Step Deployment

### 1. Create Project Directory

First, create a dedicated directory for your PrivateBin project and navigate into it:

```bash
mkdir privatebin
cd privatebin
```

### 2. Create `docker-compose.yml`

Create a `docker-compose.yml` file within this directory:

```bash
nano docker-compose.yml
```

Paste the following content into the file:

```yaml
version: '3.8'

services:
  privatebin:
    image: privatebin/privatebin:latest
    container_name: privatebin
    restart: unless-stopped
    ports:
      - "8080:80" # Map container port 80 to host port 8080. Change 8080 if it conflicts.
    volumes:
      - ./data:/srv/data # Persistent storage for pastes. Recommended to keep this.
      - ./config:/srv/cfg # Persistent configuration. Recommended.
    environment:
      # Optional: Adjust upload/post size limits if you anticipate very large pastes.
      PRIVATEBIN_PHP_INI_UPLOAD_MAX_FILESIZE: "10M"
      PRIVATEBIN_PHP_INI_POST_MAX_SIZE: "10M"
      # You can generate a custom salt for enhanced security.
      # If not set, PrivateBin will generate one automatically on first run.
      # PRIVATEBIN_SALT: "your_long_random_string_here_for_salt_generation"
    labels:
      # Example labels for a reverse proxy (e.g., Traefik, Nginx Proxy Manager)
      # If using Nginx Proxy Manager, you'd point it to http://<your_server_ip>:8080
      # If using Traefik, uncomment and adjust these labels:
      # - "traefik.enable=true"
      # - "traefik.http.routers.privatebin.rule=Host(`paste.yourdomain.com`)"
      # - "traefik.http.routers.privatebin.entrypoints=websecure"
      # - "traefik.http.routers.privatebin.tls.certresolver=myresolver" # Define your cert resolver (e.g., cloudflare, letsencrypt)
      # - "traefik.http.services.privatebin.loadbalancer.server.port=80"
```

### 3. Deploy PrivateBin

Run Docker Compose to start the PrivateBin container:

```bash
docker compose up -d
```

### 4. Access PrivateBin

PrivateBin should now be running. You can access it via your web browser:

*   **Locally:** `http://localhost:8080` (if running Docker directly on your desktop)
*   **On your server:** `http://<Your_Server_IP>:8080`

### 5. (Optional) Configure a Reverse Proxy

For secure, domain-based access (e.g., `https://paste.yourdomain.com`), it's highly recommended to set up a reverse proxy with SSL.

*   **Nginx Proxy Manager:** Create a new proxy host, pointing your domain to `http://<Your_Server_IP>:8080`. Enable SSL.
*   **Caddy:** Add a site block to your `Caddyfile`:
    ```caddy
    paste.yourdomain.com {
        reverse_proxy <Your_Server_IP>:8080
    }
    ```
    Then `caddy reload`.

### 6. Verify and Use

Open PrivateBin in your browser. You should see the simple interface ready for you to paste text. Notice how the URL includes a hash (`#`) after you paste something – this signifies client-side encryption, meaning the encrypted data is sent to the server, but the decryption key remains in your browser.

Enjoy your private, secure pastebin!
