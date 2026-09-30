### Port Listener Inspector

This powerful Bash script provides an immediate snapshot of your system's network activity, detailing all active listening TCP and UDP ports, their associated process IDs (PIDs), and the program names responsible. It's an indispensable tool for security auditing, troubleshooting connectivity issues, and ensuring no unauthorized services are running.

#### What it Does:

The script leverages the `ss` (socket statistics) utility, a modern replacement for `netstat`, to query the kernel for socket information. It then filters for listening connections (`-l`) across TCP (`-t`) and UDP (`-u`) protocols, displaying numeric addresses (`-n`) and process information (`-p`). The output is parsed to present a clean, organized view of each listening port, its protocol, local address, the PID of the process, and the program name.

#### Prerequisites:

*   **Linux/Unix-like Operating System**: The script is designed for Linux environments. It uses standard utilities available on most distributions.
*   **`ss` utility**: Part of the `iproute2` package, which is typically pre-installed on modern Linux systems. If not available, install it via your distribution's package manager (e.g., `sudo apt install iproute2` on Debian/Ubuntu, `sudo dnf install iproute2` on Fedora/RHEL).
*   **`awk` utility**: Standard text processing tool, usually pre-installed.

#### Full Code:

```bash
#!/bin/bash

# Port Listener Inspector: Identifies open ports and listening processes.

echo "Scanning for active listening TCP/UDP ports:"
echo "--------------------------------------------------"

# 'ss' is the modern utility for socket statistics.
# -t: TCP, -u: UDP, -n: Numeric, -l: Listening, -p: Processes
# This command lists all listening TCP/UDP sockets with their PID and program name.
ss -tunlp | grep -v 'Netid' | awk '
    {
        proto=$1; local_addr=$5; pid_program=$7;
        gsub(/users:\(\"|\"\)\)/, "", pid_program); # Clean up "users:((" and "))"
        split(pid_program, a, ",");
        pid="N/A"; program="N/A";
        for (i in a) {
            if (a[i] ~ /^pid=/) pid=substr(a[i], 5);
            if (a[i] ~ /^program=\"/ && length(a[i]) > 9) program=substr(a[i], 9, length(a[i])-9);
        }
        printf "%-5s %-20s %-8s %s\n", proto, local_addr, pid, program;
    }
'

echo "--------------------------------------------------"
echo "Scan complete."
```

#### Execution Commands:

1.  **Save the script**: Save the code above to a file named `port_inspector.sh`.

    ```bash
    nano port_inspector.sh
    ```

2.  **Make it executable**: Grant execute permissions to the script.

    ```bash
    chmod +x port_inspector.sh
    ```

3.  **Run the script**: Execute the script from your terminal.

    ```bash
    ./port_inspector.sh
    ```

This script is a cornerstone for any systems administrator or homelab enthusiast seeking to maintain a secure and transparent network environment.