# Netdata

Netdata is an open-source, distributed, real-time performance and health monitoring solution. It collects thousands of metrics from systems, applications, and network devices, and visualizes them in stunning, interactive dashboards. It's designed to be highly efficient, running on various systems from IoT devices to large servers, making it perfect for a home server or NAS environment.

## Prerequisites

Ensure you have Docker and Docker Compose installed.
You will need to map volumes for Netdata's configuration, data, and cache. Access to host system files like `/proc`, `/sys`, and `/var/run/docker.sock` is crucial for comprehensive monitoring.
The container will expose port `19999` for the web UI.

## Docker Compose Script

```yaml
version: '3.8'
services:
  netdata:
    image: netdata/netdata
    container_name: netdata
    hostname: ${HOSTNAME:-your-server-name} # Replace 'your-server-name' with your actual hostname
    cap_add:
      - SYS_PTRACE
      - SYS_ADMIN # Required for some collectors (e.g., cgroups, network interfaces)
    security_opt:
      - apparmor:unconfined # Required for some collectors (e.g., cgroups)
    volumes:
      - netdataconfig:/etc/netdata # Configuration files
      - netdatalib:/var/lib/netdata # Database and other runtime files
      - netdatacache:/var/cache/netdata # Cache files
      - /etc/passwd:/etc/passwd:ro # To collect user/group info
      - /etc/group:/etc/group:ro   # To collect user/group info
      - /proc:/host/proc:ro # Host /proc for system metrics
      - /sys:/host/sys:ro   # Host /sys for system metrics
      - /var/run/docker.sock:/var/run/docker.sock:ro # To monitor Docker containers
    ports:
      - "19999:19999" # Web UI
    restart: unless-stopped
    environment:
      PUID: 1000 # Optional: User ID for file permissions, adjust as needed
      PGID: 1000 # Optional: Group ID for file permissions, adjust as needed
      # NETDATA_CLAIM_TOKEN: 'YOUR_CLAIM_TOKEN' # Uncomment and add if you want to connect to Netdata Cloud
      # NETDATA_CLAIM_URL: 'https://app.netdata.cloud' # Uncomment if using Netdata Cloud
      # NETDATA_CLAIM_ROOMS: 'your_room_id' # Uncomment if using Netdata Cloud
      # NETDATA_CLAIM_PROXY: 'http://your_proxy:port' # Uncomment if using a proxy for Netdata Cloud
    labels:
      - "homepage.group=Monitoring"
      - "homepage.name=Netdata"
      - "homepage.icon=netdata.png"
      - "homepage.href=http://your-server-ip:19999"

volumes:
  netdataconfig:
  netdatalib:
  netdatacache:
```