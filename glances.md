# Glances

Glances is a cross-platform command-line monitoring tool that provides a comprehensive overview of your system's resources in real-time. It can also be accessed via a web interface or API, making it ideal for monitoring your home server or NAS.

## Prerequisites

*   A Docker-compatible host (e.g., Linux server, Synology NAS) with Docker and Docker Compose installed.
*   An available port (e.g., `61208`) on your host machine to access the web interface.
*   Understanding of Docker volumes, especially for providing necessary host system access to Glances.

## Docker Compose Script

```yaml
version: '3.8'
services:
  glances:
    image: nicolargo/glances:latest-full
    container_name: glances
    restart: unless-stopped
    pid: "host" # Required for full system monitoring of host processes
    ports:
      - "61208:61208" # Web UI access
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro # Optional: Monitor Docker containers
      - /etc/os-release:/etc/os-release:ro # Optional: Get OS information
      - /sys:/sys:ro # Required for full system monitoring (CPU, memory, etc.)
      - /run/udev:/run/udev:ro # Required for disk I/O stats on some systems
      - /var/lib/docker/containers:/var/lib/docker/containers:ro # Optional: Monitor Docker container logs
    environment:
      - GLANCES_OPT=-w # Start Glances in web server mode
      - TZ=Europe/London # Set your desired timezone (e.g., "America/New_York")
```