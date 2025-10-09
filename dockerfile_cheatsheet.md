# Dockerfile Cheatsheet

A comprehensive reference guide for Dockerfile instructions, best practices, and examples.

## 🔗 Navigation

- **[← Back to Main Docker Cheatsheet](README.md)** - Essential Docker commands
- **[Docker Compose Cheatsheet](docker-compose_cheatsheet.md)** - Multi-container orchestration

## Table of Contents

- [Introduction](#introduction)
- [Basic Instructions](#basic-instructions)
- [Advanced Instructions](#advanced-instructions)
- [Best Practices](#best-practices)
- [Common Examples](#common-examples)
- [2024-2025 Updates](#2024-2025-updates)

## Introduction

A Dockerfile is a text file that contains a series of instructions used to build Docker images automatically. It defines the environment, dependencies, and commands needed to run your application.

### Basic Structure
```dockerfile
# Comment
INSTRUCTION arguments
```

### Dockerfile Syntax Rules
- Instructions are case-insensitive (convention: UPPERCASE)
- Each instruction creates a new layer
- Instructions are executed in order
- Comments start with `#`
- Use `.dockerignore` to exclude unnecessary files
- Prefer `COPY` over `ADD` for security and clarity

## Basic Instructions

### FROM
Specifies the base image for your container.

```dockerfile
# Use official image
FROM node:18-alpine

# Use specific version
FROM ubuntu:20.04

# Use scratch (empty image)
FROM scratch

# Multi-platform
FROM --platform=linux/amd64 node:18
```

### RUN
Executes commands during image build.

```dockerfile
# Single command
RUN apt-get update

# Multiple commands (creates one layer)
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*

# Shell form
RUN echo "Hello World"

# Exec form (no shell)
RUN ["/bin/bash", "-c", "echo Hello World"]
```

### CMD
Sets the default command to run when container starts.

```dockerfile
# Shell form
CMD echo "Hello World"

# Exec form (recommended)
CMD ["echo", "Hello World"]

# Default parameters for ENTRYPOINT
CMD ["--help"]
```

### ENTRYPOINT
Sets the main command that cannot be overridden.

```dockerfile
# Shell form
ENTRYPOINT echo "Hello"

# Exec form (recommended)
ENTRYPOINT ["echo", "Hello"]

# With CMD as parameters
ENTRYPOINT ["echo"]
CMD ["Hello World"]
```

### COPY vs ADD
Copy files from host to container.

```dockerfile
# COPY (recommended for simple file copying)
COPY package.json /app/
COPY . /app/

# COPY with multiple sources
COPY package*.json ./

# ADD (has additional features)
ADD package.json /app/
ADD https://example.com/file.tar.gz /tmp/

# ADD with extraction
ADD archive.tar.gz /app/
```

### WORKDIR
Sets the working directory for subsequent instructions.

```dockerfile
# Set working directory
WORKDIR /app

# Multiple WORKDIR instructions
WORKDIR /app
WORKDIR src
# Final directory: /app/src
```

### ENV
Sets environment variables.

```dockerfile
# Single variable
ENV NODE_ENV=production

# Multiple variables
ENV NODE_ENV=production \
    PORT=3000

# Using variables
ENV NODE_ENV=production
ENV APP_HOME=/app
WORKDIR $APP_HOME
```

### EXPOSE
Documents which ports the container listens on.

```dockerfile
# Single port
EXPOSE 3000

# Multiple ports
EXPOSE 3000 8080

# With protocol
EXPOSE 3000/tcp
EXPOSE 8080/udp
```

## Advanced Instructions

### ARG
Defines build-time variables.

```dockerfile
# Define build argument
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}-alpine

# Use in build
ARG BUILD_DATE
LABEL build-date=$BUILD_DATE

# Build with: docker build --build-arg NODE_VERSION=16 .
```

### LABEL
Adds metadata to the image.

```dockerfile
# Single label
LABEL version="1.0"

# Multiple labels
LABEL version="1.0" \
      author="John Doe" \
      description="My application"

# Using variables
ARG VERSION=1.0
LABEL version=$VERSION
```

### USER
Sets the user for subsequent instructions.

```dockerfile
# Create user
RUN adduser -D appuser
USER appuser

# Use specific user
USER 1000

# Switch back to root
USER root
```

### VOLUME
Creates mount points for external volumes.

```dockerfile
# Single volume
VOLUME ["/data"]

# Multiple volumes
VOLUME ["/data", "/logs"]

# With specific mount point
VOLUME /var/lib/mysql
```

### HEALTHCHECK
Defines how to check if container is healthy.

```dockerfile
# Basic health check
HEALTHCHECK CMD curl -f http://localhost:3000/health || exit 1

# With options
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

# Disable health check
HEALTHCHECK NONE
```

### SHELL
Overrides the default shell.

```dockerfile
# Use PowerShell on Windows
SHELL ["powershell", "-command"]

# Use bash
SHELL ["/bin/bash", "-c"]
```

### ONBUILD
Triggers instructions when the image is used as base.

```dockerfile
# Trigger when used as base image
ONBUILD COPY package.json ./
ONBUILD RUN npm install
ONBUILD COPY . ./
```

## Best Practices

### Multi-stage Builds
Reduce final image size by using multiple stages.

```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
CMD ["npm", "start"]
```

### Layer Optimization
Minimize layers and optimize caching.

```dockerfile
# Bad: Multiple RUN commands
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git

# Good: Single RUN command
RUN apt-get update && \
    apt-get install -y curl git && \
    rm -rf /var/lib/apt/lists/*

# Copy package files first for better caching
COPY package*.json ./
RUN npm install
COPY . .
```

### Security Considerations

```dockerfile
# Use specific versions
FROM node:18.17.0-alpine

# Don't run as root
RUN adduser -D appuser
USER appuser

# Use .dockerignore
# Don't copy sensitive files
COPY . .
# Remove sensitive files
RUN rm -f .env.secret
```

### Image Size Optimization

```dockerfile
# Use alpine variants
FROM node:18-alpine

# Remove package manager cache
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*

# Use multi-stage builds
FROM node:18 AS builder
# ... build steps
FROM node:18-alpine
COPY --from=builder /app/dist ./dist
```

## Common Examples

### Node.js Application

```dockerfile
# Use official Node.js image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Create non-root user
RUN adduser -D appuser
USER appuser

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

# Start application
CMD ["npm", "start"]
```

### Python Application

```dockerfile
# Use official Python image
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

# Install system dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        gcc \
        && rm -rf /var/lib/apt/lists/*

# Copy requirements and install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user
RUN adduser --disabled-password --gecos '' appuser
USER appuser

# Expose port
EXPOSE 8000

# Start application
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

### Go Application

```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder

WORKDIR /app

# Copy go mod files
COPY go.mod go.sum ./

# Download dependencies
RUN go mod download

# Copy source code
COPY . .

# Build application
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Production stage
FROM alpine:latest

# Install ca-certificates for HTTPS
RUN apk --no-cache add ca-certificates

WORKDIR /root/

# Copy binary from builder stage
COPY --from=builder /app/main .

# Expose port
EXPOSE 8080

# Run application
CMD ["./main"]
```

### Multi-stage Build Example

```dockerfile
# Build stage
FROM node:18 AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install all dependencies (including dev)
RUN npm ci

# Copy source code
COPY . .

# Build application
RUN npm run build

# Production stage
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install only production dependencies
RUN npm ci --only=production && npm cache clean --force

# Copy built application from builder stage
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/public ./public

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# Change ownership
RUN chown -R nextjs:nodejs /app
USER nextjs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/api/health || exit 1

# Start application
CMD ["npm", "start"]
```

## Quick Reference

### Instruction Order (Best Practice)
1. `FROM` - Base image
2. `ARG` - Build arguments
3. `ENV` - Environment variables
4. `RUN` - Install dependencies
5. `COPY` - Copy application files
6. `WORKDIR` - Set working directory
7. `USER` - Set user (if not root)
8. `EXPOSE` - Document ports
9. `HEALTHCHECK` - Health check
10. `CMD` or `ENTRYPOINT` - Start command

### Common Patterns

```dockerfile
# Development Dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev"]

# Production Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN adduser -D appuser
USER appuser
EXPOSE 3000
CMD ["npm", "start"]
```

## 2024-2025 Updates

### Latest Best Practices
```dockerfile
# Use specific base image versions
FROM node:18.17.0-alpine

# Implement non-root user from the start
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

# Use BuildKit cache mounts
RUN --mount=type=cache,target=/root/.npm \
    npm ci --only=production

# Implement health checks
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

# Use multi-platform builds
FROM --platform=$BUILDPLATFORM node:18-alpine AS builder
```

### Security Enhancements
```dockerfile
# Use distroless images for production
FROM gcr.io/distroless/nodejs18-debian11

# Implement proper secret management
RUN --mount=type=secret,id=npm_token \
    npm config set //registry.npmjs.org/:_authToken $(cat /run/secrets/npm_token)

# Use .dockerignore for security
# .dockerignore
node_modules
.git
.env
*.log
```

### Performance Optimizations
```dockerfile
# Use BuildKit features
# syntax=docker/dockerfile:1.4

# Cache mount for package managers
RUN --mount=type=cache,target=/var/cache/apt \
    apt-get update && apt-get install -y curl

# Parallel builds
RUN --mount=type=cache,target=/root/.npm \
    npm ci --only=production
```

---

**Note**: This cheatsheet covers the most commonly used Dockerfile instructions with the latest 2024-2025 best practices. For more advanced usage, refer to the [official Docker documentation](https://docs.docker.com/engine/reference/builder/).
