### SSL Certificate Expiry Auditor

This Bash script provides a crucial utility for any sysadmin or homelab enthusiast: proactive auditing of SSL/TLS certificate expiry dates. Preventing unexpected certificate expirations is vital for maintaining service availability and security. This script allows you to quickly check the validity period for one or more domains or IP addresses, even specifying custom ports.

#### What it Does:

1.  **Connects Securely**: Uses `openssl` to establish a TLS connection to the specified host and port.
2.  **Extracts Expiry Date**: Parses the certificate information to find the `notAfter` (expiry) date.
3.  **Calculates Remaining Days**: Compares the expiry date with the current date to determine how many days are left until expiration.
4.  **Provides Warnings**: Highlights certificates that are nearing their expiration date (e.g., less than 30 days).

#### Prerequisites:

*   **Bash**: Standard on most Linux distributions and macOS.
*   **openssl**: Essential for fetching certificate details. Usually pre-installed.
*   **date**: For date calculations. Usually pre-installed.

#### Script:

```bash
#!/bin/bash

# Function to check SSL certificate expiry
check_ssl_expiry() {
    local host_port="$1"
    local host=$(echo "$host_port" | cut -d':' -f1)
    local port=$(echo "$host_port" | cut -d':' -f2)

    if [ -z "$port" ]; then
        port="443" # Default to HTTPS port
    fi

    echo "Auditing SSL for: $host:$port"

    # Use openssl to connect and extract the expiry date
    expiry_date_str=$(echo | openssl s_client -servername "$host" -connect "$host:$port" 2>/dev/null | \
                     openssl x509 -noout -enddate | cut -d'=' -f2)

    if [ -z "$expiry_date_str" ]; then
        echo "  [✗] Failed to retrieve certificate for $host:$port"
        return
    fi

    # Convert expiry date to Unix timestamp
    expiry_timestamp=$(date -d "$expiry_date_str" +%s)
    current_timestamp=$(date +%s)

    # Calculate days remaining
    days_remaining=$(( (expiry_timestamp - current_timestamp) / 86400 ))

    echo "  [✓] Expiry Date: $expiry_date_str"
    echo "  [→] Days Remaining: $days_remaining days"

    if [ "$days_remaining" -lt 30 ]; then
        echo "  [!] WARNING: Certificate expires soon!"
    fi
}

# Main execution
if [ -z "$1" ]; then
    echo "Usage: $0 <host:port> [host2:port2 ...]"
    echo "Example: $0 example.com my.service.net:8443"
    exit 1
fi

# Loop through all provided targets
for target in "$@"; do
    check_ssl_expiry "$target"
    echo "--------------------------"
done
```

#### Execution:

1.  **Save the script**: Save the code above to a file, e.g., `ssl_check.sh`.
2.  **Make it executable**: `chmod +x ssl_check.sh`
3.  **Run with target(s)**:

    ```bash
    ./ssl_check.sh example.com my.secure.app:8443 another.domain.org
    ```

    The script will output the expiry details for each specified host and port, providing immediate insight into your certificate health.