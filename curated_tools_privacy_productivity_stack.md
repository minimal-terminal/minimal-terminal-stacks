# The Ultimate Self-Hosted Privacy & Productivity Stack

In the pursuit of digital sovereignty, a carefully curated stack of open-source tools is paramount. This selection empowers you to reclaim your data, enhance productivity, and fortify your personal infrastructure with minimal overhead. Each tool is chosen for its robustness, community support, and alignment with privacy-first principles.

---

## The Stack

### 01. Homer
**Description:** A simple, static, responsive dashboard for your self-hosted services. Easily configurable with YAML, Homer provides a clean, centralized access point to all your applications.
**Project Link:** [https://github.com/bastienwirtz/homer](https://github.com/bastienwirtz/homer)
**Docker Compose:**
```yaml
version: "3.3"
services:
  homer:
    image: ghcr.io/bastienwirtz/homer:latest
    container_name: homer
    volumes:
      - ./homer_config:/app/assets
    ports:
      - "8080:8080"
    restart: unless-stopped
```

### 02. Plausible Analytics
**Description:** A lightweight, open-source, and privacy-friendly alternative to Google Analytics. Plausible provides all the essential website traffic insights without tracking cookies or collecting personal data.
**Project Link:** [https://plausible.io/self-hosted](https://plausible.io/self-hosted)
**Docker Compose:**
```yaml
version: "3.6"
services:
  plausible_db:
    image: postgres:13-alpine
    restart: unless-stopped
    volumes:
      - plausible_db_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: your_strong_password
      POSTGRES_USER: plausible
      POSTGRES_DB: plausible
  plausible:
    image: plausible/analytics:latest
    container_name: plausible
    restart: unless-stopped
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://plausible:your_strong_password@plausible_db:5432/plausible
      BASE_URL: https://analytics.yourdomain.com
      SECRET_KEY_BASE: generate_with_mix_phx_gen_secret
    depends_on:
      - plausible_db
volumes:
  plausible_db_data:
```

### 03. Whoogle Search
**Description:** A self-hosted, ad-free, privacy-respecting metasearch engine. Whoogle fetches search results from Google without ads, AMP links, or IP logging, providing a clean and private search experience.
**Project Link:** [https://github.com/benbusby/whoogle-search](https://github.com/benbusby/whoogle-search)
**Docker Compose:**
```yaml
version: "3.8"
services:
  whoogle:
    image: ghcr.io/benbusby/whoogle-search:latest
    container_name: whoogle
    ports:
      - "5000:5000"
    environment:
      WHOOGLE_CONFIG_ALLOW_METRICS: "False" # Optional: disable metrics
    restart: unless-stopped
```

### 04. Wallabag
**Description:** An open-source, self-hostable "read-it-later" application. Wallabag allows you to save web pages for offline reading, archiving, and easy access across devices, free from ads and distractions.
**Project Link:** [https://www.wallabag.org/](https://www.wallabag.org/)
**Docker Compose:**
```yaml
version: "3.8"
services:
  wallabag:
    image: wallabag/wallabag:latest
    container_name: wallabag
    environment:
      - SYMFONY__ENV__DATABASE_DRIVER=pdo_sqlite
      - SYMFONY__ENV__DATABASE_PATH=/var/www/wallabag/data/db/wallabag.sqlite
      - SYMFONY__ENV__DOMAIN_NAME=https://wallabag.yourdomain.com
    volumes:
      - ./wallabag_data:/var/www/wallabag/data
    ports:
      - "80:80" # Map to a different port if 80 is in use
    restart: unless-stopped
```

### 05. Joplin Server
**Description:** A synchronization server for Joplin notes, allowing you to self-host your secure, end-to-end encrypted notes across multiple devices without relying on third-party cloud services.
**Project Link:** [https://joplinapp.org/help/server/](https://joplinapp.org/help/server/)
**Docker Compose:**
```yaml
version: "3.8"
services:
  joplin_db:
    image: postgres:13-alpine
    restart: unless-stopped
    volumes:
      - joplin_db_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: your_strong_db_password
      POSTGRES_USER: joplin
      POSTGRES_DB: joplin
  joplin_server:
    image: joplin/server:latest
    container_name: joplin_server
    restart: unless-stopped
    ports:
      - "22300:22300"
    depends_on:
      - joplin_db
    environment:
      APP_PORT: 22300
      DB_CLIENT: pg
      POSTGRES_PASSWORD: your_strong_db_password
      POSTGRES_DATABASE: joplin
      POSTGRES_USER: joplin
      POSTGRES_HOST: joplin_db
      POSTGRES_PORT: 5432
      # You must generate a secret for JWT
      # NODE_ENV=production npm run generate-jwt-secret
      # Or just provide a long random string
      APP_SECRET: your_long_random_jwt_secret
volumes:
  joplin_db_data:
```

### 06. Apache Guacamole
**Description:** An HTML5 web application that provides clientless remote desktop access. Guacamole allows you to connect to your RDP, SSH, and VNC sessions directly from your web browser, securely and conveniently.
**Project Link:** [https://guacamole.apache.org/](https://guacamole.apache.org/)
**Docker Compose:**
```yaml
version: "3.8"
services:
  guacd:
    image: guacamole/guacd:latest
    container_name: guacd
    restart: unless-stopped
  guacamole:
    image: guacamole/guacamole:latest
    container_name: guacamole
    restart: unless-stopped
    links:
      - guacd:guacd
    environment:
      GUACD_HOSTNAME: guacd
      # Optional: AUTHENTICATION_PROVIDER=org.apache.guacamole.auth.jdbc.mysql.MySQLAuthenticationProvider
      # For database authentication, you'd need a MySQL/PostgreSQL container and configure it.
    ports:
      - "8080:8080" # Adjust port as needed
```

### 07. Prowlarr
**Description:** A comprehensive indexer manager for Sonarr, Radarr, Lidarr, and Readarr. Prowlarr integrates with your download clients and indexers to automate media acquisition and management.
**Project Link:** [https://prowlarr.com/](https://prowlarr.com/)
**Docker Compose:**
```yaml
version: "3.8"
services:
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - ./prowlarr_config:/config
    ports:
      - "9696:9696"
    restart: unless-stopped
```

### 08. Tandoor Recipes
**Description:** A self-hosted recipe management system designed for home cooks. Organize your recipes, plan meals, generate shopping lists, and discover new dishes with a beautiful, feature-rich interface.
**Project Link:** [https://tandoor.recipes/](https://tandoor.recipes/)
**Docker Compose:**
```yaml
version: "3.8"
services:
  tandoor:
    image: vabene1111/tandoor-recipes:latest
    container_name: tandoor
    environment:
      - DJANGO_SETTINGS_MODULE=tandoor.settings.production
      - DB_ENGINE=django.db.backends.sqlite3
      - DB_NAME=/app/data/db.sqlite3
      - SECRET_KEY=your_long_random_secret_key
      - ALLOWED_HOSTS=yourdomain.com,localhost,127.0.0.1
      - MEDIA_ROOT=/app/data/media
      - STATIC_ROOT=/app/data/static
      - EMAIL_HOST=
      - EMAIL_PORT=
      - EMAIL_HOST_USER=
      - EMAIL_HOST_PASSWORD=
      - DEFAULT_FROM_EMAIL=
      - SERVER_EMAIL=
    volumes:
      - ./tandoor_data:/app/data
    ports:
      - "8000:8000"
    restart: unless-stopped
```