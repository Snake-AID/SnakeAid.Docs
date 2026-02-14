---
doc_role: baseline
module: docker
kind: layer
status: active
last_updated: 2026-02-15
owners: [backend-team]
---

# Docker - Source Code

## Docker Compose
**Location**: [docker-compose.yml](../../docker-compose.yml)

Configuration for running the SnakeAid Backend service.

```yaml
services:
  snakeaid-backend:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=http://+:8080
    restart: unless-stopped
```

## Dockerfile
**Location**: [Dockerfile](../../Dockerfile)

(See [Jenkins Source Code](../Jenkins/jenkins.sourcecode.md) for Dockerfile content)
