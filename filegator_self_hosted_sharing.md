# Self-Hosted Secure File Transfer & Sharing with Filegator

This blueprint guides you through deploying Filegator, a powerful and flexible open-source file manager, for secure file transfer and sharing within your homelab or on a dedicated server. Reclaim control over your data and provide a private alternative to cloud-based sharing services.

## Prerequisites

*   **Linux Server:** A machine running a modern Linux distribution (e.g., Ubuntu, Debian, Fedora).
*   **Docker:** Docker Engine installed and running on your server.
*   **Docker Compose:** Docker Compose installed (usually comes with Docker Desktop or needs separate installation for server environments).
*   **SSH Access:** Ability to connect to your server via SSH.

## Step 1: Prepare Your Environment

First, create a dedicated directory for your Filegator project and a subdirectory for persistent data. This ensures your configurations, user data, and uploaded files are preserved even if the container is recreated.

```bash
# Create the main project directory
mkdir -p ~/filegator
cd ~/filegator

# Create a volume directory for Filegator's data
mkdir -p ./data
```

## Step 2: Deploy Filegator with Docker Compose

Create a `docker-compose.yml` file in your `~/filegator` directory. This configuration will define the Filegator service, its persistent storage, and port mappings.

```yaml
version: '3.8'

services:
  filegator:
    image: filegator/filegator
    container_name: filegator
    restart: unless-stopped
    ports:
      - "8080:80" # Host_Port:Container_Port - Access Filegator via http://your_server_ip:8080
    volumes:
      - ./data:/var/www/html/data # Persistent storage for users, config, and uploaded files
      # - ./config/configuration.php:/var/www/html/configuration.php:ro # Optional: Mount a custom configuration file
      # - ./files:/var/www/html/repository # Optional: Mount an external directory as a Filegator repository
    environment:
      # Optional: Set initial admin password (change immediately after first login)
      # FILEGATOR_ADMIN_PASSWORD: "your_strong_admin_password" 
      # FILEGATOR_BASEURL: "http://your_domain.com" # If behind a reverse proxy
      TZ: "America/New_York" # Set your timezone
```

Save the above content as `docker-compose.yml` in your `~/filegator` directory.

Now, deploy Filegator by running Docker Compose:

```bash
docker compose up -d
```

This command will download the Filegator image (if not already present), create the container, and start it in detached mode.

## Step 3: Initial Setup and Configuration

Once the container is running, you can access Filegator through your web browser.

1.  **Access Filegator:** Open your web browser and navigate to `http://your_server_ip:8080`.
2.  **Initial Login:** The default credentials are `admin` for the username and `admin123` for the password. **It is CRITICAL to change this immediately after your first login.**
3.  **Change Admin Password:** After logging in, navigate to the `Users` section (usually accessible via the top-right menu or sidebar), select the `admin` user, and set a strong, unique password.
4.  **Create New Users & Groups:** For secure sharing, create separate user accounts for different individuals or purposes. You can assign them to groups and define specific permissions for each user/group regarding file access, uploads, and downloads.
5.  **Configure Sharing Options:** Explore Filegator's sharing features. You can create public or password-protected share links for files and folders, setting expiry dates or download limits.

## Step 4: Secure Your Access (Basic Considerations)

For a basic homelab setup, consider these points for security:

*   **Firewall Rules:** If exposed to the internet, ensure your server's firewall (`ufw`, `firewalld`) only allows access to port `8080` from trusted IPs or a reverse proxy. For internal use, you might restrict access to your local network.
*   **Strong Passwords:** Always use strong, unique passwords for all Filegator user accounts.
*   **Keep Updated:** Regularly update your Docker images and underlying server OS to patch security vulnerabilities. You can update Filegator by pulling the latest image and recreating the container:
    ```bash
    docker compose pull filegator
    docker compose up -d
    ```

For production environments or internet exposure, it is highly recommended to place Filegator behind a reverse proxy (e.g., Nginx, Apache) with SSL/TLS encryption for secure communication.