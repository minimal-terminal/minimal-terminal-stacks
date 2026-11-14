# Automated Network Device Inventory with Nmap and Python

## Overview
Elevate your homelab's security and management with a custom-built, automated network device inventory system. This project combines the robust scanning capabilities of Nmap with Python scripting to systematically discover, identify, and log every device on your local network. You'll gain insights into IP addresses, MAC addresses, hostnames, detected operating systems, and open ports, all stored in a persistent SQLite database for easy auditing and reference. This blueprint empowers you to maintain a clear, up-to-date record of your network's topology, crucial for security, troubleshooting, and compliance.

## Key Milestones
1.  **Network Mapping**: Utilize Nmap's powerful features to perform comprehensive port and OS detection across your specified network range.
2.  **Data Persistence**: Develop a Python script to parse Nmap's XML output and store the structured device data (IP, MAC, hostname, OS, open ports) in a lightweight SQLite database.
3.  **Scheduled Scans**: Configure a Dockerized solution that can be easily integrated with external schedulers (like `cron` or `systemd` timers) for automated, regular updates of your network inventory.

## Prerequisites
-   **Docker & Docker Compose**: Installed and running on your host system (e.g., a Raspberry Pi, NAS, or Linux server).
-   **Basic Linux Terminal Knowledge**: Familiarity with `cd`, `ls`, `nano` or `vi`, and `docker compose` commands.
-   **Network Range**: Know your local network's CIDR range (e.g., `192.168.1.0/24`).

## Project Setup

First, create a project directory and navigate into it:
```bash
mkdir network-inventory-project
cd network-inventory-project
```

### 1. Create the Python Script (`scan_network.py`)

This script orchestrates the Nmap scan, parses its XML output, and stores the results in an SQLite database.

```python
import subprocess
import xml.etree.ElementTree as ET
import sqlite3
import os
import datetime
import sys

# Configuration
NETWORK_RANGE = os.getenv('NETWORK_RANGE', '192.168.1.0/24')
DB_PATH = os.getenv('DB_PATH', '/data/network_inventory.db')

def init_db():
    """Initializes the SQLite database and creates the 'devices' table if it doesn't exist."""
    try:
        conn = sqlite3.connect(DB_PATH)
        cursor = conn.cursor()
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS devices (
                ip_address TEXT PRIMARY KEY,
                mac_address TEXT,
                hostname TEXT,
                os_guess TEXT,
                open_ports TEXT,
                last_seen TEXT
            )
        ''')
        conn.commit()
        conn.close()
        print(f"[*] Database initialized at {DB_PATH}")
    except sqlite3.Error as e:
        print(f"[-] Error initializing database: {e}", file=sys.stderr)
        sys.exit(1)

def run_nmap_scan(network_range):
    """Executes an Nmap scan and returns the XML output."""
    print(f"[*] Starting Nmap scan for {network_range}...")
    try:
        # -sS (SYN scan), -O (OS detection), -p 1-1000 (common ports), -T4 (faster), -oX (XML output)
        # Using -oX - sends XML to stdout
        result = subprocess.run(
            ['nmap', '-sS', '-O', '-p', '1-1000', '-T4', '-oX', '-', network_range],
            capture_output=True, text=True, check=True, timeout=300 # 5 minute timeout
        )
        print("[+] Nmap scan complete.")
        return result.stdout
    except subprocess.CalledProcessError as e:
        print(f"[-] Nmap scan failed with error code {e.returncode}: {e}", file=sys.stderr)
        print(f"Stderr: {e.stderr}", file=sys.stderr)
        return None
    except subprocess.TimeoutExpired:
        print(f"[-] Nmap scan timed out after 300 seconds.", file=sys.stderr)
        return None
    except FileNotFoundError:
        print(f"[-] Nmap command not found. Ensure Nmap is installed in the container.", file=sys.stderr)
        sys.exit(1)


def parse_nmap_xml(xml_data):
    """Parses Nmap XML output and extracts device information."""
    devices = []
    if not xml_data:
        return devices
    
    try:
        root = ET.fromstring(xml_data)
    except ET.ParseError as e:
        print(f"[-] Error parsing Nmap XML data: {e}", file=sys.stderr)
        return devices

    for host in root.findall('host'):
        ip_address_elem = host.find('address[@addrtype="ipv4"]')
        if ip_address_elem is None:
            continue
        ip_address = ip_address_elem.get('addr')

        mac_address_elem = host.find('address[@addrtype="mac"]')
        mac_address = mac_address_elem.get('addr') if mac_address_elem is not None else 'N/A'

        hostname_elem = host.find('hostnames/hostname')
        hostname = hostname_elem.get('name') if hostname_elem is not None else 'N/A'

        os_guess = 'N/A'
        os_match = host.find('os/osmatch')
        if os_match is not None:
            os_guess = os_match.get('name')

        open_ports = []
        for port in host.findall('ports/port'):
            state = port.find('state')
            if state is not None and state.get('state') == 'open':
                service = port.find('service')
                service_name = service.get('name') if service is not None else 'unknown'
                open_ports.append(f"{port.get('portid')}/{port.get('protocol')} ({service_name})")
        
        devices.append({
            'ip_address': ip_address,
            'mac_address': mac_address,
            'hostname': hostname,
            'os_guess': os_guess,
            'open_ports': ', '.join(open_ports),
            'last_seen': datetime.datetime.now().isoformat()
        })
    return devices

def update_db(devices):
    """Updates the SQLite database with new or updated device information."""
    try:
        conn = sqlite3.connect(DB_PATH)
        cursor = conn.cursor()
        for device in devices:
            cursor.execute('''
                INSERT OR REPLACE INTO devices (ip_address, mac_address, hostname, os_guess, open_ports, last_seen)
                VALUES (?, ?, ?, ?, ?, ?)
            ''', (
                device['ip_address'],
                device['mac_address'],
                device['hostname'],
                device['os_guess'],
                device['open_ports'],
                device['last_seen']
            ))
        conn.commit()
        conn.close()
        print(f"[+] Database updated with {len(devices)} devices.")
    except sqlite3.Error as e:
        print(f"[-] Error updating database: {e}", file=sys.stderr)
        sys.exit(1)

def print_inventory():
    """Prints the current network inventory from the database."""
    print("\n[*] Current Network Inventory:")
    try:
        conn = sqlite3.connect(DB_PATH)
        cursor = conn.cursor()
        cursor.execute('SELECT ip_address, hostname, mac_address, os_guess, open_ports, last_seen FROM devices ORDER BY ip_address')
        rows = cursor.fetchall()
        if not rows:
            print("  No devices found in inventory.")
            return

        for row in rows:
            print(f"  IP: {row[0]}, Hostname: {row[1]}, MAC: {row[2]}, OS: {row[3]}, Ports: [{row[4]}], Last Seen: {row[5]}")
        conn.close()
    except sqlite3.Error as e:
        print(f"[-] Error reading inventory from database: {e}", file=sys.stderr)
        sys.exit(1)


if __name__ == "__main__":
    init_db()
    xml_output = run_nmap_scan(NETWORK_RANGE)
    if xml_output:
        parsed_devices = parse_nmap_xml(xml_output)
        update_db(parsed_devices)
    
    print_inventory()
    print("[*] Network inventory process finished.")

```

### 2. Create the Dockerfile

This `Dockerfile` builds a lightweight image containing Python and Nmap, ready to execute your script.

```dockerfile
FROM python:3.9-slim-buster

LABEL maintainer="Your Name <your.email@example.com>"

# Install Nmap and its dependencies
# -qq suppresses most output, --no-install-recommends avoids unnecessary packages
RUN apt-get update -qq && \
    apt-get install -yqq --no-install-recommends nmap && \
    rm -rf /var/lib/apt/lists/*

# Set working directory inside the container
WORKDIR /app

# Copy the Python script into the container
COPY scan_network.py .

# Create a directory for the SQLite database
RUN mkdir -p /data

# Set the default command to run the Python script
CMD ["python", "scan_network.py"]
```

### 3. Create the Docker Compose File (`docker-compose.yml`)

This file defines your service, mounts the necessary volumes, and sets the network configuration for Nmap to function correctly.

```yaml
version: '3.8'

services:
  network-inventory:
    build:
      context: . # Build from the current directory
      dockerfile: Dockerfile
    container_name: network-inventory
    restart: "no" # Set to 'on-failure' or 'unless-stopped' for persistent automation
    
    # Nmap requires raw socket access, which is only possible with host networking
    # or specific capabilities. 'host' network mode is simplest for full network access.
    network_mode: "host" 
    
    # Mount a host directory to persist the SQLite database
    volumes:
      - ./data:/data 
    
    # Environment variables for the Python script
    environment:
      - NETWORK_RANGE=192.168.1.0/24 # IMPORTANT: CHANGE THIS TO YOUR LOCAL NETWORK RANGE (e.g., 192.168.1.0/24, 10.0.0.0/24)
      - DB_PATH=/data/network_inventory.db
      
    # Grant necessary capabilities for Nmap's raw socket operations (e.g., SYN scan)
    # These are needed if not using 'host' network mode, but good practice to include.
    cap_add:
      - NET_ADMIN
      - NET_RAW
    
    # The command to run the script is already defined in the Dockerfile's CMD.
    # If you want to override it, you can uncomment and modify this line:
    # command: python scan_network.py
```

### 4. Run the Project

Ensure you are in the `network-inventory-project` directory.

1.  **Build and Run (First Time):**
    This command will build the Docker image, create the `network-inventory` container, and run your script.
    ```bash
    docker compose up --build --force-recreate
    ```
    You will see Nmap output, followed by the script's database update messages and the final inventory summary.

2.  **Run Subsequent Scans:**
    After the initial run, you can simply restart the container to perform a new scan.
    ```bash
    docker compose up
    ```
    Or, if you want to run it once and exit:
    ```bash
    docker compose run --rm network-inventory
    ```

### 5. Inspect the Inventory

The SQLite database file `network_inventory.db` will be located in the `data/` subdirectory of your project. You can inspect it using a SQLite browser or `sqlite3` CLI.

Example using `sqlite3` CLI (if installed on your host):
```bash
sqlite3 data/network_inventory.db "SELECT * FROM devices;"
```

### 6. Automation with `cron` or `systemd` (Optional)

For automated, regular scans, you can use `cron` or `systemd` timers on your host system.

**Using `cron`:**
Edit your crontab:
```bash
crontab -e
```
Add a line to run the scan daily at 3 AM (adjust path to your project directory):
```
0 3 * * * cd /path/to/your/network-inventory-project && /usr/local/bin/docker compose up > /dev/null 2>&1
```
*Note: Ensure `docker compose` is in your PATH or use the full path to the executable.*

**Using `systemd` Timer (Recommended for robust scheduling):**
Create two files in `/etc/systemd/system/`:

`network-inventory.service`:
```ini
[Unit]
Description=Run Network Inventory Scan
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
WorkingDirectory=/path/to/your/network-inventory-project
ExecStart=/usr/local/bin/docker compose up
StandardOutput=journal
StandardError=journal
```
*Remember to replace `/path/to/your/network-inventory-project` and `/usr/local/bin/docker compose` with the correct paths.*

`network-inventory.timer`:
```ini
[Unit]
Description=Schedule Network Inventory Scan Daily

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

Enable and start the timer:
```bash
sudo systemctl enable network-inventory.timer
sudo systemctl start network-inventory.timer
sudo systemctl status network-inventory.timer
```

This project provides a foundational tool for understanding and securing your home network, giving you a detailed, automated inventory of all connected devices.