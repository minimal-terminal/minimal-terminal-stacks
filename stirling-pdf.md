# Stirling-PDF

Stirling-PDF is a powerful, open-source, locally hosted web application that allows you to perform various operations on PDF files. It provides a clean, modern interface for tasks like merging, splitting, rotating, reordering, compressing, and converting PDFs, all while keeping your data private on your own server.

## Prerequisites

Ensure you have Docker and Docker Compose installed on your system.
A dedicated volume for Stirling-PDF data is recommended for persistence.
Choose an available port (e.g., 8080) for the web interface.

## Docker Compose Script

```yaml
version: '3.8'
services:
  stirling-pdf:
    image: ghcr.io/stirling-tools/stirling-pdf:latest
    container_name: stirling-pdf
    ports:
      - "8080:8080" # Host_Port:Container_Port
    volumes:
      - ./stirling-pdf-data:/app/data # Optional: For persistent settings/uploads if needed
    environment:
      - DOCKER_ENABLE_SECURITY=true # Recommended for production
      - ENABLE_WEBDAV=false # Disable if not using WebDAV
      - TZ=America/New_York # Set your timezone
    restart: unless-stopped
```