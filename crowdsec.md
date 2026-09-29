### CrowdSec: Collaborative Security for Your Homelab

CrowdSec is an open-source, lightweight, and collaborative Intrusion Prevention System (IPS) and Intrusion Detection System (IDS). It analyzes logs from your servers and services (SSH, web servers, databases, etc.) to detect malicious behaviors. When a threat is identified, CrowdSec can automatically block the attacker using various "bouncers" that integrate with your firewall, reverse proxy, or other infrastructure components. By joining the CrowdSec network, you also benefit from community-sourced threat intelligence, protecting your systems from IPs already identified as malicious by others.

---

#### Prerequisites

Before deploying CrowdSec, ensure you have:

*   **Docker and Docker Compose:** Installed and configured on your host system.
*   **Host Log Access:** CrowdSec needs read access to your host's log files (e.g., `/var/log/auth.log`, `/var/log/nginx/access.log`). These directories will be mounted into the container.
*   **Basic Linux Knowledge:** Familiarity with the terminal for interacting with the CrowdSec CLI (`cscli`).

---

#### Docker Compose Stack

This stack deploys the core CrowdSec agent. Once running, you can configure it to ingest logs and install bouncers to enforce blocking decisions.

```yaml
version: '3.8'

services:
  crowdsec:
    image: crowdsec/crowdsec:latest
    container_name: crowdsec
    restart: unless-stopped
    environment:
      # Set your desired timezone (e.g., "America/New_York", "Europe/Paris")
      TZ: "Etc/UTC" 
    volumes:
      # Persistent storage for CrowdSec configurations, database, and logs
      - ./crowdsec-data:/etc/crowdsec 
      # Mount host log directories for CrowdSec to scan. Adjust paths as needed.
      - /var/log:/var/log:ro 
      # Optional: Mount Docker volumes for container logs if you use Docker's logging drivers
      # - /var/lib/docker/volumes:/var/lib/docker/volumes:ro
    networks:
      - crowdsec-net
    # Expose the LAPI (Local API) if you plan to have bouncers or other CrowdSec instances
    # connect to this agent from outside the Docker network.
    # ports:
    #   - "8080:8080" 

networks:
  crowdsec-net:
    driver: bridge
```

---

#### Post-Deployment & Usage

1.  **Deploy the Stack:**
    Navigate to your desired deployment directory and save the above content as `docker-compose.yml`. Then run:
    ```bash
    docker compose up -d
    ```

2.  **Access the CLI:**
    You can interact with CrowdSec using its CLI (`cscli`) inside the container:
    ```bash
    docker exec -it crowdsec cscli
    ```
    Use `cscli metrics` to see real-time statistics, `cscli decisions list` to view active blocking decisions, and `cscli alerts list` for detected threats.

3.  **Install Collections:**
    CrowdSec uses "collections" to define parsers and scenarios for specific software (e.g., Nginx, SSH). Install relevant collections for your services:
    ```bash
    docker exec -it crowdsec cscli collections install crowdsecurity/nginx crowdsecurity/ssh-openvpn
    docker exec -it crowdsec cscli parsers install crowdsecurity/syslog-logs
    docker exec -it crowdsec cscli scenarios install crowdsecurity/ssh-bf crowdsecurity/http-bf
    ```
    *Note: `crowdsecurity/ssh-openvpn` is a collection that includes parsers and scenarios for SSH logs. `crowdsecurity/nginx` is for Nginx logs. You may need to adapt these based on your actual services.*

4.  **Install Bouncers (Enforcement):**
    The agent detects threats; bouncers enforce the blocking. Bouncers are typically installed separately on the host or as dedicated containers, connecting to the CrowdSec agent's Local API (LAPI).
    *   **Firewall Bouncer (Recommended for Host Protection):** For blocking at the host firewall level. Often installed directly on the host machine.
        ```bash
        # Example for Debian/Ubuntu (run on host, not in CrowdSec container)
        curl -s https://packagecloud.io/install/repositories/crowdsec/crowdsec/script.deb.sh | sudo bash
        sudo apt install crowdsec-firewall-bouncer
        sudo systemctl enable crowdsec-firewall-bouncer
        sudo systemctl start crowdsec-firewall-bouncer
        ```
        The bouncer will automatically register with your Dockerized CrowdSec agent if they are on the same network or if the LAPI is exposed and configured.
    *   **Nginx Bouncer:** To protect web services behind Nginx.
        Refer to the official CrowdSec documentation for integrating bouncers with your specific setup.

CrowdSec provides a robust, community-driven security layer, transforming your homelab into a more resilient and secure environment.
