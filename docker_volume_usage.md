# Docker Volume Usage Summarizer

This Bash script provides a quick and efficient way to audit the disk space consumed by all Docker volumes on your host system. It's an invaluable tool for sysadmins and homelabbers looking to manage storage, identify resource hogs, and maintain a clean Docker environment.

## Prerequisites

*   **Docker:** The Docker daemon must be running, and the `docker` CLI tool must be installed and accessible.
*   **Bash:** A standard Bash environment.
*   **sudo:** The script uses `sudo` for `du` to ensure it can read all volume paths, even those with restrictive permissions. Ensure your user has `sudo` privileges without password prompt for `du`, or be prepared to enter your password.

## Script: `docker_volume_usage.sh`

```bash
#!/bin/bash
# Get disk usage for all Docker volumes
echo "--- Docker Volume Disk Usage ---"
echo "SIZE\tVOLUME_NAME"
echo "--------------------------------"

docker volume ls -q | while read -r vol; do
    path=$(docker volume inspect -f '{{ .Mountpoint }}' "$vol")
    # Check if the mount path exists and get its size, otherwise mark as N/A
    [ -d "$path" ] && size=$(sudo du -sh "$path" 2>/dev/null | awk '{print $1}') || size="N/A"
    echo -e "${size:-"N/A"}\t$vol"
done
echo "--------------------------------"
```

## How to Use

1.  **Save the script:**
    Create a new file named `docker_volume_usage.sh` on your system and paste the script content into it.

    ```bash
    nano docker_volume_usage.sh
    # Paste content, then Ctrl+S, Ctrl+X
    ```

2.  **Make it executable:**
    Grant execution permissions to the script.

    ```bash
    chmod +x docker_volume_usage.sh
    ```

3.  **Run the script:**
    Execute the script from your terminal.

    ```bash
    ./docker_volume_usage.sh
    ```

    The script will output a summary table showing the disk usage for each Docker volume. If a volume's mount path cannot be found or accessed, it will be listed as "N/A".

## Example Output

```
--- Docker Volume Disk Usage ---
SIZE    VOLUME_NAME
--------------------------------
4.0K    my_app_data
12G     plex_config
8.3G    nextcloud_data
2.5G    db_data
N/A     orphan_volume
--------------------------------
```

This summary helps you quickly identify which volumes are consuming the most space, enabling better resource management and troubleshooting.
