### Subnet IP Scanner

This minimalist Bash script provides a quick and efficient way to scan a `/24` subnet for active hosts. It iterates through all possible IP addresses within the specified prefix and uses a single `ping` request to determine if a host is online.

#### What it Does

- Takes a subnet prefix (e.g., `192.168.1`) as an argument.
- Loops from `1` to `254` to construct full IP addresses.
- Sends a single `ping` packet to each IP with a 1-second timeout.
- Reports whether the host is `UP` if it responds.

#### Prerequisites

- **Operating System**: Linux or macOS.
- **Utilities**: `bash` (standard on most systems) and `ping`.

#### Script Code

```bash
#!/bin/bash
# Simple Subnet Scanner - Discover active hosts quickly.

SUBNET_PREFIX="$1"
if [ -z "$SUBNET_PREFIX" ]; then
    echo "Usage: $0 <subnet_prefix (e.g., 192.168.1)>"
    exit 1
fi

echo "[+] Scanning hosts on ${SUBNET_PREFIX}.0/24..."

for i in $(seq 1 254); do
    IP="${SUBNET_PREFIX}.$i"
    ping -c 1 -W 1 "$IP" > /dev/null 2>&1
    if [ $? -eq 0 ]; then
        echo "[+] $IP : UP"
    fi
done

echo "[+] Scan complete for ${SUBNET_PREFIX}.0/24."
```

#### Execution

1.  **Save the script**: Save the code above to a file named `subnet_scan.sh`.
2.  **Make it executable**: 
    ```bash
    chmod +x subnet_scan.sh
    ```
3.  **Run the script**: Provide the first three octets of your desired subnet.
    ```bash
    ./subnet_scan.sh 192.168.1
    ```
    (Replace `192.168.1` with your actual network prefix.)

This script is ideal for quick network reconnaissance and troubleshooting on your homelab or local network.