# Docker Cheatsheet

A comprehensive Docker reference guide with essential commands and concepts.

## 📚 Related Cheatsheets

- **[Dockerfile Cheatsheet](dockerfile_cheatsheet.md)** - Complete guide to Dockerfile instructions and best practices
- **[Docker Compose Cheatsheet](docker-compose_cheatsheet.md)** - Multi-container application orchestration

## Table of Contents

- [Introduction](#introduction)
- [Installation & Setup](#installation--setup)
- [Image Commands](#image-commands)
- [Container Commands](#container-commands)
- [Docker Compose](#docker-compose)
- [Networking](#networking)
- [Volumes & Storage](#volumes--storage)
- [Security Best Practices](#security-best-practices)
- [Other Useful Commands](#other-useful-commands)

## Introduction

Docker is a containerization platform that allows you to package applications and their dependencies into lightweight, portable containers. This cheatsheet covers the most commonly used Docker commands and concepts, updated with the latest features and best practices for 2024-2025.

### Key Features (2024-2025 Updates)
- **Enhanced Security**: Latest security patches and best practices
- **Docker Build Early Access**: Cloud-based build acceleration
- **Improved Multi-stage Builds**: Better layer optimization
- **Advanced Networking**: Enhanced container communication
- **Resource Management**: Better CPU and memory controls

## Installation & Setup

### Verify Installation
```bash
# Check Docker version
docker --version

# Check Docker info
docker info

# Test Docker installation
docker run hello-world
```

### Docker System Information
```bash
# System-wide information
docker system info

# Disk usage
docker system df

# Show Docker events
docker events
```

## Image Commands

### Basic Image Operations
```bash
# Pull an image
docker pull <image_name>
docker pull nginx:latest

# List images
docker images
docker image ls

# Remove an image
docker rmi <image_id>
docker rmi nginx:latest

# Remove all unused images
docker image prune
```

### Building Images
```bash
# Build from Dockerfile
docker build -t <image_name> .
docker build -t myapp:latest .

# Build with specific Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# Build with build arguments
docker build --build-arg VERSION=1.0 -t myapp:1.0 .

# Build with BuildKit (faster builds)
DOCKER_BUILDKIT=1 docker build -t myapp:latest .

# Build with cache mount (2024 feature)
docker build --build-arg BUILDKIT_INLINE_CACHE=1 -t myapp:latest .

# Build for multiple platforms
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest .
```

### Image Management
```bash
# Tag an image
docker tag <source_image> <target_image>
docker tag nginx:latest myregistry/nginx:v1.0

# Push to registry
docker push <image_name>
docker push myregistry/nginx:v1.0

# Save image to file
docker save -o myimage.tar <image_name>

# Load image from file
docker load -i myimage.tar

# Inspect image details
docker inspect <image_name>
```

## Container Commands

### Running Containers
```bash
# Run a container
docker run <image_name>
docker run nginx

# Run with name
docker run --name mycontainer nginx

# Run in background (detached)
docker run -d nginx

# Run with port mapping
docker run -p 8080:80 nginx

# Run with environment variables
docker run -e ENV_VAR=value nginx

# Run with volume mount
docker run -v /host/path:/container/path nginx

# Run with interactive terminal
docker run -it ubuntu bash

# Run with security options (2024 best practice)
docker run --read-only --tmpfs /tmp nginx

# Run with user namespace (security)
docker run --userns=host nginx

# Run with resource limits
docker run --memory=512m --cpus=1.0 nginx

# Run with health check
docker run --health-cmd="curl -f http://localhost:3000/health" nginx
```

### Container Management
```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Start a container
docker start <container_id>

# Stop a container
docker stop <container_id>

# Restart a container
docker restart <container_id>

# Remove a container
docker rm <container_id>

# Remove all stopped containers
docker container prune
```

### Container Operations
```bash
# Execute command in running container
docker exec <container_id> <command>
docker exec mycontainer ls -la

# Execute interactive shell
docker exec -it <container_id> bash

# View container logs
docker logs <container_id>

# Follow logs in real-time
docker logs -f <container_id>

# View container stats
docker stats

# Inspect container details
docker inspect <container_id>
```

## Docker Compose

### Basic Commands
```bash
# Start services
docker-compose up

# Start in background
docker-compose up -d

# Build and start
docker-compose up --build

# Stop services
docker-compose down

# View logs
docker-compose logs

# Follow logs
docker-compose logs -f

# Scale services
docker-compose up --scale web=3
```

### Service Management
```bash
# Start specific service
docker-compose up <service_name>

# Stop services
docker-compose stop

# Restart services
docker-compose restart

# Remove services
docker-compose rm

# Execute command in service
docker-compose exec <service_name> <command>
```

## Networking

### Network Commands
```bash
# List networks
docker network ls

# Create network
docker network create <network_name>
docker network create mynetwork

# Inspect network
docker network inspect <network_name>

# Remove network
docker network rm <network_name>

# Connect container to network
docker network connect <network_name> <container_name>

# Disconnect container from network
docker network disconnect <network_name> <container_name>
```

### Container Networking
```bash
# Run container with specific network
docker run --network <network_name> <image_name>

# Run with host networking
docker run --network host <image_name>

# Run with custom network
docker run --network mynetwork --name container1 nginx
```

## Volumes & Storage

### Volume Commands
```bash
# List volumes
docker volume ls

# Create volume
docker volume create <volume_name>
docker volume create myvolume

# Inspect volume
docker volume inspect <volume_name>

# Remove volume
docker volume rm <volume_name>

# Remove unused volumes
docker volume prune
```

### Mounting Volumes
```bash
# Named volume
docker run -v myvolume:/app/data nginx

# Bind mount
docker run -v /host/path:/container/path nginx

# Read-only mount
docker run -v /host/path:/container/path:ro nginx

# Multiple mounts
docker run -v vol1:/data1 -v vol2:/data2 nginx
```

## Security Best Practices

### Container Security
```bash
# Run container as non-root user
docker run --user 1000:1000 nginx

# Use read-only filesystem
docker run --read-only --tmpfs /tmp nginx

# Set security options
docker run --security-opt no-new-privileges nginx

# Use seccomp profile
docker run --security-opt seccomp=seccomp-profile.json nginx

# Limit capabilities
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE nginx
```

### Image Security
```bash
# Scan image for vulnerabilities
docker scan nginx:latest

# Use specific image tags (avoid 'latest')
docker run nginx:1.21-alpine

# Sign images with Docker Content Trust
export DOCKER_CONTENT_TRUST=1
docker push myregistry/myapp:1.0

# Use multi-stage builds to reduce attack surface
# See Dockerfile Cheatsheet for examples
```

### Network Security
```bash
# Use custom networks for isolation
docker network create --driver bridge isolated-network

# Run containers on isolated networks
docker run --network isolated-network nginx

# Use internal networks (no external access)
docker network create --internal internal-network
```

## Other Useful Commands

### Cleanup Commands
```bash
# Remove all stopped containers
docker container prune

# Remove all unused images
docker image prune

# Remove all unused volumes
docker volume prune

# Remove all unused networks
docker network prune

# Remove everything unused
docker system prune

# Remove everything including volumes
docker system prune -a --volumes
```

### System Information
```bash
# Docker version
docker version

# System information
docker system info

# Disk usage
docker system df

# Show running processes
docker top <container_id>

# Copy files to/from container
docker cp <container_id>:/path/to/file /host/path
docker cp /host/path <container_id>:/path/to/file
```

### Advanced Commands
```bash
# Commit container to image
docker commit <container_id> <new_image_name>

# Export container to tar
docker export <container_id> > container.tar

# Import from tar
docker import container.tar <image_name>

# Show container resource usage
docker stats <container_id>

# Update container restart policy
docker update --restart=unless-stopped <container_id>
```

## Common Flags Reference

| Flag | Description |
|------|-------------|
| `-d` | Run in detached mode (background) |
| `-it` | Interactive terminal |
| `-p` | Port mapping (host:container) |
| `-v` | Volume mount |
| `-e` | Environment variable |
| `--name` | Container name |
| `--rm` | Remove container when it stops |
| `--restart` | Restart policy |
| `-f` | Follow logs output |
| `-a` | All containers/images |
| `-q` | Quiet mode (IDs only) |

## Quick Reference

### Most Used Commands
```bash
# Start development environment
docker-compose up -d

# View logs
docker-compose logs -f

# Stop everything
docker-compose down

# Clean up
docker system prune

# Run one-off command
docker run --rm -it <image> <command>
```

---

**Note**: This cheatsheet covers the most commonly used Docker commands. For more advanced usage, refer to the [official Docker documentation](https://docs.docker.com/).
