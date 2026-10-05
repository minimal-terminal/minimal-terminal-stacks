### The Essential Self-Hosted Stack

Elevate your homelab with these powerful, open-source applications. Each tool is chosen for its utility, privacy focus, and ease of deployment via Docker.

---

#### 01. Plex Media Server

*   **Description**: The ultimate media server for your movies, TV shows, music, and photos. Stream to any device, anywhere, with a beautifully organized interface.
*   **Link**: [https://www.plex.tv/](https://www.plex.tv/)
*   **Docker Run Command**:
    ```bash
    docker run \
      -d \
      --name=plex \
      --network=host \
      -e PUID=1000 \
      -e PGID=1000 \
      -e VERSION=docker \
      -v /path/to/plex/config:/config \
      -v /path/to/tvseries:/tv \
      -v /path/to/movies:/movies \
      --restart unless-stopped \
      lscr.io/linuxserver/plex:latest
    ```
    *Note: Adjust PUID/PGID to your user ID and group ID (`id -u` and `id -g`). Replace `/path/to/...` with your actual host paths.*

---

#### 02. Heimdall Application Dashboard

*   **Description**: A clean, minimalist dashboard for all your self-hosted applications. Easily access your services with a beautiful, customizable interface.
*   **Link**: [https://heimdall.site/](https://heimdall.site/)
*   **Docker Run Command**:
    ```bash
    docker run \
      -d \
      --name=heimdall \
      -e PUID=1000 \
      -e PGID=1000 \
      -e TZ=Etc/UTC \
      -p 80:80 \
      -p 443:443 \
      -v /path/to/heimdall/config:/config \
      --restart unless-stopped \
      lscr.io/linuxserver/heimdall:latest
    ```
    *Note: Adjust PUID/PGID and TZ. Map ports carefully if 80/443 are in use.*

---

#### 03. Whoogle Search

*   **Description**: A self-hosted, ad-free, privacy-respecting metasearch engine that fetches results from Google without ads, JavaScript, or IP tracking.
*   **Link**: [https://github.com/benbusby/whoogle-search](https://github.com/benbusby/whoogle-search)
*   **Docker Run Command**:
    ```bash
    docker run \
      -d \
      --name=whoogle-search \
      -e WHOOGLE_CONFIG_HOSTNAME=0.0.0.0 \
      -p 5000:5000 \
      --restart unless-stopped \
      ghcr.io/benbusby/whoogle-search:latest
    ```
    *Note: Access Whoogle via `http://your-server-ip:5000`.*

---

#### 04. Miniflux

*   **Description**: A minimalist and opinionated RSS feed reader. Fast, efficient, and designed for focused, distraction-free reading.
*   **Link**: [https://miniflux.app/](https://miniflux.app/)
*   **Docker Run Command**:
    ```bash
    docker run \
      -d \
      --name=miniflux \
      -e DATABASE_URL="sqlite:///data/db.sqlite" \
      -e LISTEN_ADDR="0.0.0.0:8080" \
      -v /path/to/miniflux/data:/data \
      -p 8080:8080 \
      --restart unless-stopped \
      miniflux/miniflux:latest
    ```
    *Note: This configuration uses SQLite for simplicity. Replace `/path/to/miniflux/data` for persistent storage.*

---

#### 05. Ntfy

*   **Description**: Send push notifications to your phone or desktop via simple HTTP PUT/POST requests. Reliable, customizable, and open-source.
*   **Link**: [https://ntfy.sh/](https://ntfy.sh/)
*   **Docker Run Command**:
    ```bash
    docker run \
      -d \
      --name=ntfy \
      -p 80:80 \
      -p 443:443 \
      -v /path/to/ntfy/cache:/var/cache/ntfy \
      -v /path/to/ntfy/config:/etc/ntfy \
      --restart unless-stopped \
      binwiederpur/ntfy:latest
    ```
    *Note: Requires configuration in `/path/to/ntfy/config/server.yml` for full features (e.g., authentication, topics, TLS).*

---

#### 06. Changedetection.io

*   **Description**: Monitor websites for changes. Get notified when content, prices, or availability updates on any web page. Essential for tracking crucial information.
*   **Link**: [https://changedetection.io/](https://changedetection.io/)
*   **Docker Run Command**:
    ```bash
    docker run \
      -d \
      --name=changedetection \
      -p 5000:5000 \
      -v /path/to/changedetection/datastore:/datastore \
      --restart unless-stopped \
      dgtlmoon/changedetection.io:latest
    ```
    *Note: Access via `http://your-server-ip:5000`. Persistent data stored in `/path/to/changedetection/datastore`.*

---

#### 07. Diun (Docker Image Update Notifier)

*   **Description**: Get notified when a new Docker image is released for your running containers. A proactive approach to keeping your services up-to-date.
*   **Link**: [https://crazy-max.github.io/diun/](https://crazy-max.github.io/diun/)
*   **Docker Run Command**:
    ```bash
    docker run \
      -d \
      --name=diun \
      -v /var/run/docker.sock:/var/run/docker.sock:ro \
      -v /path/to/diun/data:/data \
      --restart unless-stopped \
      crazymax/diun:latest
    ```
    *Note: Configuration for notification services (e.g., email, Discord) goes into `/path/to/diun/data/diun.yml`.*

---

#### 08. Komga

*   **Description**: A self-hosted media server for your comics, mangas, and magazines. Enjoy your digital collection with a beautiful web reader.
*   **Link**: [https://komga.org/](https://komga.org/)
*   **Docker Run Command**:
    ```bash
    docker run \
      -d \
      --name=komga \
      -p 8080:8080 \
      -v /path/to/komga/config:/config \
      -v /path/to/komga/data:/data \
      -v /path/to/comics:/comics \
      --restart unless-stopped \
      ghcr.io/komga/komga:latest
    ```
    *Note: Replace `/path/to/comics` with the actual path to your comic library.*

---

#### 09. Shiori

*   **Description**: A simple yet powerful bookmark manager written in Go. Organize your links with tags and easily search through them, keeping your web finds private.
*   **Link**: [https://github.com/go-shiori/shiori](https://github.com/go-shiori/shiori)
*   **Docker Run Command**:
    ```bash
    docker run \
      -d \
      --name=shiori \
      -p 8080:8080 \
      -v /path/to/shiori/data:/data \
      --restart unless-stopped \
      ghcr.io/go-shiori/shiori:latest
    ```
    *Note: Access via `http://your-server-ip:8080`. Your bookmarks are stored in `/path/to/shiori/data`.*
