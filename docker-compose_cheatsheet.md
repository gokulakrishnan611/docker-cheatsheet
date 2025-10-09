# Docker Compose Cheatsheet

A comprehensive reference guide for Docker Compose configuration, commands, and best practices.

## 🔗 Navigation

- **[← Back to Main Docker Cheatsheet](README.md)** - Essential Docker commands
- **[Dockerfile Cheatsheet](dockerfile_cheatsheet.md)** - Dockerfile instructions and best practices

## Table of Contents

- [Introduction](#introduction)
- [Basic Configuration](#basic-configuration)
- [Advanced Configuration](#advanced-configuration)
- [Docker Compose CLI Commands](#docker-compose-cli-commands)
- [Common Patterns](#common-patterns)
- [Best Practices](#best-practices)
- [Real-world Examples](#real-world-examples)
- [2024-2025 Updates](#2024-2025-updates)

## Introduction

Docker Compose is a tool for defining and running multi-container Docker applications. It uses YAML files to configure your application's services, networks, and volumes.

### docker-compose.yml Structure
```yaml
# Note: version field is deprecated in Compose v2
services:
  web:
    image: nginx
    ports:
      - "80:80"

volumes:
  data:

networks:
  frontend:
```

### Version Differences (2024-2025)
- **Compose v2**: Current standard, no version field needed
- **Legacy v3.8**: Still supported but deprecated
- **New Format**: Uses Compose Specification without version field

## Basic Configuration

### version (Deprecated in Compose v2)
Specifies the Compose file format version.

```yaml
# Deprecated in Compose v2 - no longer needed
# version: '3.8'  # Legacy format
# version: '3.7'  # Legacy format
# version: '2.4'  # Legacy format

# New Compose v2 format (no version field)
services:
  web:
    image: nginx
```

### services
Defines the containers that make up your application.

```yaml
services:
  web:
    image: nginx:alpine
    container_name: my-web
    ports:
      - "8080:80"
  
  database:
    image: postgres:13
    container_name: my-db
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
```

### image
Specifies the Docker image to use.

```yaml
services:
  web:
    image: nginx:alpine
    # or
    image: myregistry/myapp:latest
```

### build
Builds an image from a Dockerfile.

```yaml
services:
  web:
    build: .
    # or with context
    build:
      context: ./app
      dockerfile: Dockerfile.prod
    # or with build args
    build:
      context: .
      args:
        NODE_ENV: production
```

### ports
Maps container ports to host ports.

```yaml
services:
  web:
    ports:
      - "3000:3000"        # host:container
      - "8080:80"          # Different ports
      - "127.0.0.1:3000:3000"  # Bind to specific interface
      - "3000"             # Random host port
```

### volumes
Mounts host directories or named volumes.

```yaml
services:
  web:
    volumes:
      - ./app:/app                    # Bind mount
      - /var/lib/mysql:/var/lib/mysql # Host path
      - data:/app/data                # Named volume
      - type: volume, source: data, target: /app/data  # Long syntax
```

### environment
Sets environment variables.

```yaml
services:
  web:
    environment:
      - NODE_ENV=production
      - DEBUG=true
    # or
    environment:
      NODE_ENV: production
      DEBUG: true
    # or from file
    env_file:
      - .env
      - .env.local
```

### networks
Defines custom networks.

```yaml
services:
  web:
    networks:
      - frontend
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true
```

## Advanced Configuration

### depends_on
Defines service dependencies.

```yaml
services:
  web:
    depends_on:
      - database
      - redis
  
  database:
    image: postgres:13
  
  redis:
    image: redis:alpine
```

### restart policies
Controls container restart behavior.

```yaml
services:
  web:
    restart: no              # Never restart
    restart: always          # Always restart
    restart: on-failure      # Restart on failure
    restart: unless-stopped  # Restart unless manually stopped
```

### healthcheck
Defines health checks for services.

```yaml
services:
  web:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### deploy (for swarm)
Configuration for Docker Swarm deployment.

```yaml
services:
  web:
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

### logging
Configures logging for services.

```yaml
services:
  web:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

### secrets and configs
Manages sensitive data and configuration files.

```yaml
services:
  web:
    secrets:
      - db_password
    configs:
      - app_config

secrets:
  db_password:
    file: ./secrets/db_password.txt

configs:
  app_config:
    file: ./config/app.conf
```

### extends
Inherits configuration from another service.

```yaml
services:
  web:
    extends:
      file: common.yml
      service: webapp
```

## Docker Compose CLI Commands

### Basic Commands

```bash
# Start services
docker-compose up

# Start in background
docker-compose up -d

# Start specific services
docker-compose up web database

# Stop services
docker-compose down

# Stop and remove volumes
docker-compose down -v

# Stop and remove images
docker-compose down --rmi all
```

### Build Commands

```bash
# Build services
docker-compose build

# Build specific service
docker-compose build web

# Build without cache
docker-compose build --no-cache

# Build and start
docker-compose up --build
```

### Service Management

```bash
# Start services
docker-compose start

# Stop services
docker-compose stop

# Restart services
docker-compose restart

# Restart specific service
docker-compose restart web

# Scale services
docker-compose up --scale web=3
```

### Information Commands

```bash
# List running services
docker-compose ps

# View logs
docker-compose logs

# Follow logs
docker-compose logs -f

# View logs for specific service
docker-compose logs web

# Execute command in service
docker-compose exec web bash

# Run one-time command
docker-compose run web npm install
```

### Configuration Commands

```bash
# Validate configuration
docker-compose config

# Show configuration
docker-compose config --services

# Pull images
docker-compose pull

# Push images
docker-compose push
```

## Common Patterns

### Web Application Stack

```yaml
version: '3.8'

services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend
    environment:
      - REACT_APP_API_URL=http://backend:5000

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    depends_on:
      - database
    environment:
      - DATABASE_URL=postgresql://user:password@database:5432/myapp
    volumes:
      - ./backend:/app

  database:
    image: postgres:13
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Development Environment

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
    command: npm run dev

  database:
    image: postgres:13
    environment:
      - POSTGRES_DB=myapp_dev
      - POSTGRES_USER=dev
      - POSTGRES_PASSWORD=dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_dev_data:/var/lib/postgresql/data

volumes:
  postgres_dev_data:
```

### Multi-service Application

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    depends_on:
      - web
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf

  web:
    build: .
    environment:
      - DATABASE_URL=postgresql://user:password@database:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - database
      - redis

  database:
    image: postgres:13
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

## Best Practices

### Environment Variables

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    image: myapp:latest
    environment:
      - NODE_ENV=${NODE_ENV:-production}
      - DATABASE_URL=${DATABASE_URL}
    env_file:
      - .env
```

```bash
# .env file
NODE_ENV=production
DATABASE_URL=postgresql://user:password@localhost:5432/myapp
```

### Override Files

```yaml
# docker-compose.override.yml (for development)
version: '3.8'

services:
  web:
    build: .
    volumes:
      - .:/app
    command: npm run dev
    ports:
      - "3000:3000"
```

```yaml
# docker-compose.prod.yml (for production)
version: '3.8'

services:
  web:
    image: myapp:latest
    restart: always
    environment:
      - NODE_ENV=production
```

### Networking Strategies

```yaml
version: '3.8'

services:
  web:
    networks:
      - frontend
      - backend

  api:
    networks:
      - backend
      - database

  database:
    networks:
      - database

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
  database:
    driver: bridge
    internal: true
```

### Volume Management

```yaml
version: '3.8'

services:
  web:
    volumes:
      - app_data:/app/data
      - ./config:/app/config:ro

  database:
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./backups:/backups

volumes:
  app_data:
    driver: local
  postgres_data:
    driver: local
```

## Real-world Examples

### LAMP Stack

```yaml
version: '3.8'

services:
  web:
    image: php:8.0-apache
    ports:
      - "80:80"
    volumes:
      - ./src:/var/www/html
      - ./apache-config:/etc/apache2/sites-available
    depends_on:
      - database
    environment:
      - MYSQL_HOST=database

  database:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=rootpassword
      - MYSQL_DATABASE=myapp
      - MYSQL_USER=user
      - MYSQL_PASSWORD=password
    volumes:
      - mysql_data:/var/lib/mysql
    ports:
      - "3306:3306"

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    ports:
      - "8080:80"
    environment:
      - PMA_HOST=database
    depends_on:
      - database

volumes:
  mysql_data:
```

### MERN Stack

```yaml
version: '3.8'

services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:5000
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongo:27017/myapp
      - JWT_SECRET=your-secret-key
    depends_on:
      - mongo

  mongo:
    image: mongo:5.0
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    volumes:
      - mongo_data:/data/db
    ports:
      - "27017:27017"

  mongo-express:
    image: mongo-express
    ports:
      - "8081:8081"
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_URL=mongodb://admin:password@mongo:27017/
    depends_on:
      - mongo

volumes:
  mongo_data:
```

### Microservices Setup

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - api-gateway

  api-gateway:
    build: ./api-gateway
    ports:
      - "3000:3000"
    environment:
      - USER_SERVICE_URL=http://user-service:3001
      - ORDER_SERVICE_URL=http://order-service:3002
    depends_on:
      - user-service
      - order-service

  user-service:
    build: ./user-service
    ports:
      - "3001:3001"
    environment:
      - DATABASE_URL=postgresql://user:password@user-db:5432/users
    depends_on:
      - user-db

  order-service:
    build: ./order-service
    ports:
      - "3002:3002"
    environment:
      - DATABASE_URL=postgresql://user:password@order-db:5432/orders
    depends_on:
      - order-db

  user-db:
    image: postgres:13
    environment:
      - POSTGRES_DB=users
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - user_db_data:/var/lib/postgresql/data

  order-db:
    image: postgres:13
    environment:
      - POSTGRES_DB=orders
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - order_db_data:/var/lib/postgresql/data

volumes:
  user_db_data:
  order_db_data:
```

## Quick Reference

### Essential Commands
```bash
# Start development environment
docker-compose up -d

# View logs
docker-compose logs -f

# Stop everything
docker-compose down

# Rebuild and start
docker-compose up --build

# Scale services
docker-compose up --scale web=3

# Execute command
docker-compose exec web bash
```

### Common Configuration Patterns
```yaml
# Development override
version: '3.8'
services:
  web:
    volumes:
      - .:/app
    command: npm run dev

# Production configuration
version: '3.8'
services:
  web:
    restart: always
    environment:
      - NODE_ENV=production
```

## 2024-2025 Updates

### Compose v2 Changes
```yaml
# New Compose v2 format (no version field)
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    environment:
      - NODE_ENV=production
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

### Enhanced Security Features
```yaml
services:
  web:
    image: nginx:alpine
    security_opt:
      - no-new-privileges:true
    read_only: true
    tmpfs:
      - /tmp
    user: "1000:1000"
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
```

### Resource Management
```yaml
services:
  web:
    image: nginx:alpine
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
```

### Environment Management
```yaml
# Use .env files for sensitive data
services:
  web:
    image: nginx:alpine
    env_file:
      - .env
      - .env.local
    environment:
      - NODE_ENV=${NODE_ENV:-production}
      - DATABASE_URL=${DATABASE_URL}
```

### Health Checks
```yaml
services:
  web:
    image: nginx:alpine
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

---

**Note**: This cheatsheet covers the most commonly used Docker Compose features with the latest 2024-2025 updates. For more advanced usage, refer to the [official Docker Compose documentation](https://docs.docker.com/compose/).
