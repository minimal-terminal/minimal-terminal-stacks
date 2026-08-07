# Actual Budget

Actual Budget is a powerful, privacy-focused, and local-first budgeting application that allows you to manage your finances using the zero-based budgeting methodology (similar to YNAB). Self-hosting Actual Budget gives you complete control over your financial data, ensuring privacy and security while providing a robust platform for tracking income, expenses, and setting financial goals.

## Prerequisites

Before deploying Actual Budget, ensure you have Docker and Docker Compose installed on your system.
A dedicated volume named `actual_budget_data` will be created to persist your application's data, ensuring your budget information is safe across container restarts and updates.
The application will be accessible via port `5006` on your host machine.

## Docker Compose Script

```yaml
version: '3.8'

services:
  actual-budget:
    image: actualbudget/actual-server:latest
    container_name: actual-budget
    restart: unless-stopped
    ports:
      - "5006:5006"
    volumes:
      - actual_budget_data:/app/data # Persist application data

volumes:
  actual_budget_data:
```