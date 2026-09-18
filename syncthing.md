# Syncthing

Syncthing is an open-source, peer-to-peer file synchronization application that lets you keep your files in sync across multiple devices, securely and privately. It eliminates the need for a central server, ensuring your data remains under your control.

## Prerequisites

Before deploying Syncthing, ensure you have:
-   Docker and Docker Compose installed on your system.
-   A dedicated directory on your host for Syncthing's configuration (e.g., `/path/to/syncthing/config`).
-   One or more directories on your host that you intend to synchronize (e.g., `/path/to/syncthing/data`).
-   Appropriate `PUID` (User ID) and `PGID` (Group ID) for the user that will own the files. You can typically find these using `id -u youruser` and `id -g youruser` on your Linux host.

## Docker Compose Script

```yaml
version: "2.1"
services:
  syncthing:
    image: lscr.io/linuxserver/syncthing:latest
    container_name: syncthing
    hostname: your-syncthing-hostname # Optional: Set a unique hostname for this Syncthing instance
    environment:
      - PUID=1000 # Replace with your User ID
      - PGID=1000 # Replace with your Group ID
      - TZ=Etc/UTC # Replace with your local timezone (e.g., Europe/London, America/New_York)
    volumes:
      - /path/to/syncthing/config:/config # IMPORTANT: Replace with the actual path for Syncthing's configuration
      - /path/to/syncthing/data:/data # IMPORTANT: Replace with the actual path for your primary synced folder
      # Uncomment and modify the line below to add additional folders to sync:
      # - /path/to/another/synced/folder:/path/in/container/for/another/folder_name_in_syncthing
    ports:
      - 8384:8384/tcp # Web UI for management (access via http://your-server-ip:8384)
      - 22000:22000/tcp # TCP port for synchronization with other devices
      - 22000:22000/udp # UDP port for synchronization with other devices
      - 21027:21027/udp # UDP port for discovery of other Syncthing instances
    restart: unless-stopped
    # network_mode: host # Uncomment this line if you experience issues with device discovery or NAT traversal,
                      # especially when syncing between Docker containers on the same host or with devices
                      # outside your local network without proper port forwarding.
```