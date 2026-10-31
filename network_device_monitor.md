# Self-Hosted Network Device Availability Monitor with Telegram Alerts

## Introduction
Never be caught off guard by a downed server or network device again. This weekend, build a lean, self-hosted monitoring solution that pings your critical devices and sends instant Telegram notifications when their status changes. Simple, effective, and entirely within your control.

## Prerequisites
*   A Linux server (e.g., Raspberry Pi, mini-PC, or a VM)
*   Basic Linux command-line familiarity
*   `ping` utility (usually pre-installed)
*   `curl` for sending Telegram messages
*   `jq` for JSON parsing (`sudo apt install jq`)
*   A Telegram Bot Token and your Chat ID (instructions below)

## Project Blueprint

### Step 1: Set up Your Telegram Bot
1.  **Create a new Bot**: Open Telegram, search for `@BotFather`, and send `/newbot`. Follow the instructions to choose a name and username for your bot. `@BotFather` will give you an **HTTP API Token**. Save this token securely.
2.  **Get your Chat ID**: Start a chat with your new bot. Then, in your browser, go to `https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates` (replace `<YOUR_BOT_TOKEN>` with your actual token). Look for the `"chat":{"id":...}` field. This is your **Chat ID**. Save this ID.

### Step 2: Prepare Your Project Directory
Create a dedicated directory for your project:
```bash
mkdir ~/network-monitor && cd ~/network-monitor
```

### Step 3: Create Configuration File (`config.sh`)
This file will store your sensitive Telegram credentials and the list of devices to monitor.
```bash
#!/bin/bash

# Telegram Bot API Token (from BotFather)
TELEGRAM_BOT_TOKEN="YOUR_TELEGRAM_BOT_TOKEN" # <-- CHANGE THIS

# Your Telegram Chat ID (from getUpdates)
TELEGRAM_CHAT_ID="YOUR_TELEGRAM_CHAT_ID"     # <-- CHANGE THIS

# List of devices to monitor (IP addresses or hostnames)
# Format: "DEVICE_NAME:IP_OR_HOSTNAME"
declare -A DEVICES=(
    ["Router"]="192.168.1.1"
    ["NAS"]="192.168.1.10"
    ["Home Assistant"]="192.168.1.20"
    ["Pi-hole"]="192.168.1.25"
    ["Google DNS"]="8.8.8.8" # Example external host
)

# File to store previous states
STATE_FILE="${SCRIPT_DIR}/network_monitor_state.json"
```
**Remember to replace `YOUR_TELEGRAM_BOT_TOKEN` and `YOUR_TELEGRAM_CHAT_ID` with your actual values.**

### Step 4: Create the Monitoring Script (`monitor_devices.sh`)
This Bash script will perform the pings, check states, and send notifications.

```bash
#!/bin/bash

# Source configuration
SCRIPT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )"
source "${SCRIPT_DIR}/config.sh"

# Function to send Telegram message
send_telegram_message() {
    local message="$1"
    echo "Sending Telegram message: $message"
    curl -s -X POST \
         "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
         -d "chat_id=${TELEGRAM_CHAT_ID}" \
         -d "text=${message}" \
         -d "parse_mode=Markdown" > /dev/null
}

# Load previous states
load_state() {
    if [ -f "$STATE_FILE" ]; then
        cat "$STATE_FILE"
    else
        echo "{}"
    fi
}

# Save current states
save_state() {
    echo "$1" > "$STATE_FILE"
}

# Main monitoring logic
monitor_devices() {
    local previous_states_json=$(load_state)
    local current_states_json="{}"

    for device_name in "${!DEVICES[@]}"; do
        local ip_hostname="${DEVICES[$device_name]}"
        local status="OFFLINE"

        # Ping the device 3 times with a 1-second timeout
        if ping -c 3 -W 1 "$ip_hostname" &> /dev/null; then
            status="ONLINE"
        fi

        # Extract previous status from JSON
        local previous_status=$(echo "$previous_states_json" | jq -r ".["$device_name"] // \"UNKNOWN\"")

        # Check for status change
        if [ "$status" != "$previous_status" ]; then
            if [ "$previous_status" == "UNKNOWN" ]; then
                echo "Initial status for ${device_name} (${ip_hostname}): ${status}"
            elif [ "$status" == "ONLINE" ]; then
                send_telegram_message "✅ *${device_name}* (${ip_hostname}) is now *ONLINE*."
                echo "${device_name} (${ip_hostname}) changed from ${previous_status} to ${status}. Notification sent."
            else # status == "OFFLINE"
                send_telegram_message "❌ *ALERT*: *${device_name}* (${ip_hostname}) is now *OFFLINE*!"
                echo "${device_name} (${ip_hostname}) changed from ${previous_status} to ${status}. Notification sent."
            fi
        fi

        # Update current states
        current_states_json=$(echo "$current_states_json" | jq ".["$device_name"] = \"$status\"")
    done

    save_state "$current_states_json"
    echo "Monitoring cycle complete. State saved."
}

# Check if jq is installed
if ! command -v jq &> /dev/null; then
    echo "jq is not installed. Please install it: sudo apt install jq"
    exit 1
fi

monitor_devices
```
Make the script executable:
```bash
chmod +x monitor_devices.sh
```

### Step 5: Test the Script
Run the script manually to ensure it works and sends test notifications:
```bash
./monitor_devices.sh
```
You should see output in your terminal and potentially Telegram messages if a device's status is "UNKNOWN" initially (it will report its current status).

### Step 6: Automate with Systemd Service
For robust, continuous operation, we'll create a systemd service that runs the script periodically.

1.  **Create the systemd service file**:
    ```bash
sudo nano /etc/systemd/system/network-monitor.service
    ```
    Add the following content:
    ```ini
[Unit]
Description=Network Device Availability Monitor
After=network-online.target

[Service]
ExecStart=/bin/bash -c "cd /home/YOUR_USERNAME/network-monitor && ./monitor_devices.sh"
WorkingDirectory=/home/YOUR_USERNAME/network-monitor
StandardOutput=journal
StandardError=journal
Restart=on-failure
User=YOUR_USERNAME # <-- CHANGE THIS to your actual username
Group=YOUR_USERNAME # <-- CHANGE THIS to your actual username

[Install]
WantedBy=multi-user.target
    ```
    **Remember to replace `YOUR_USERNAME` with your actual Linux username.**

2.  **Create the systemd timer file**: This will tell systemd how often to run the service.
    ```bash
sudo nano /etc/systemd/system/network-monitor.timer
    ```
    Add the following content:
    ```ini
[Unit]
Description=Run Network Device Availability Monitor every 5 minutes

[Timer]
OnBootSec=1min
OnUnitActiveSec=5min # Run every 5 minutes after the unit becomes active
AccuracySec=1min     # Allow some flexibility in execution time

[Install]
WantedBy=timers.target
    ```

3.  **Enable and Start the Timer**:
    ```bash
sudo systemctl daemon-reload
sudo systemctl enable network-monitor.timer
sudo systemctl start network-monitor.timer
    ```

4.  **Check Status**:
    ```bash
systemctl status network-monitor.timer
systemctl status network-monitor.service
journalctl -f -u network-monitor.service
    ```
    The `journalctl` command will show you the logs from your script.

## Conclusion
You've successfully deployed a self-hosted network device availability monitor! This simple yet powerful tool will keep you informed about the status of your critical homelab infrastructure, giving you peace of mind and proactive alerts. Customize the `config.sh` with more devices and enjoy your enhanced network visibility!
