# Scrutiny

Scrutiny is a WebUI for smartd data, designed to make it easy to monitor the health of your hard drives and SSDs. It collects SMART data from your drives, visualizes it, and provides alerts for potential issues, ensuring the longevity and reliability of your storage devices.

## Prerequisites

Before deploying Scrutiny, ensure you have:

*   **Docker and Docker Compose:** Installed on your system.
*   **Persistent Storage:** A dedicated directory for Scrutiny's configuration and internal database.
*   **Device Access:** Scrutiny needs direct access to your storage devices (`/dev/sdX` or `/dev/nvmeXn1`). You will need to map these devices into the container or provide broader `/dev` access (less recommended for security).
*   **Open Port:** An available port on your host (e.g., `8080`) for the web interface.

## Docker Compose Script

```yaml
version: '3.8'

services:
  scrutiny:
    image: ghcr.io/analogj/scrutiny:master-collector
    container_name: scrutiny
    ports:
      - "8080:8080" # Web UI
    volumes:
      - ./scrutiny/config:/opt/scrutiny/config # Scrutiny configuration and database
      - ./scrutiny/influxdb:/opt/scrutiny/influxdb # Internal InfluxDB data
      - /run/udev:/run/udev:ro # Required for device discovery
      # Map your specific drives. Replace /dev/sdX with your actual drive paths.
      # You can find your drive paths using `ls -l /dev/disk/by-id` or `lsblk`.
      # Example for a SATA drive:
      - /dev/sda:/dev/sda:rwm
      # Example for an NVMe drive:
      # - /dev/nvme0n1:/dev/nvme0n1:rwm
      # Add more drives as needed:
      # - /dev/sdb:/dev/sdb:rwm
      # - /dev/sdc:/dev/sdc:rwm
    cap_add:
      - SYS_RAWIO # Required for SMART data access
    environment:
      TZ: Etc/UTC # Set your timezone, e.g., 'America/New_York'
    restart: unless-stopped
    # For some systems, you might need to add `privileged: true` if `cap_add` and `devices` are insufficient.
    # However, `cap_add: SYS_RAWIO` and explicit device mappings are generally preferred over `privileged: true`.
```