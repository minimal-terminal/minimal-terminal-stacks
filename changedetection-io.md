## Changedetection.io: Your Digital Sentinel

Changedetection.io is a powerful, open-source tool that monitors websites for changes. It's an indispensable asset for tracking price drops, policy updates, news, or any dynamic web content, providing alerts when a specified element or the entire page changes.

### Prerequisites

- Docker and Docker Compose installed.

```yaml
version: '3.8'
services:
  changedetection:
    image: dgtlmoon/changedetection.io
    container_name: changedetection
    restart: unless-stopped
    ports:
      - "5000:5000" # Or your desired host port, e.g., "8080:5000"
    volumes:
      - ./changedetection-data:/datastore
    environment:
      - PUID=1000 # Replace with your user ID
      - PGID=1000 # Replace with your group ID
      - TZ=Etc/UTC # Replace with your timezone, e.g., Europe/London
    networks:
      - changedetection_net

networks:
  changedetection_net:
    driver: bridge
```