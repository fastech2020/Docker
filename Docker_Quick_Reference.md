# Docker Quick Reference & Troubleshooting Guide

## Quick Start

### Installation
- **Windows/Mac**: Download Docker Desktop from https://www.docker.com/products/docker-desktop
- **Linux**: `curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh`

### Verify Installation
```bash
docker --version
docker run hello-world
```

---

## Essential Workflow

### 1. Build an Image
```bash
# Create a Dockerfile
cat > Dockerfile << EOF
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
EOF

# Build the image
docker build -t myapp:1.0 .
```

### 2. Run a Container
```bash
# Basic run
docker run myapp:1.0

# Run in background with port mapping
docker run -d -p 8080:80 --name myapp myapp:1.0

# Run with environment variables
docker run -d -e ENV_VAR=value myapp:1.0

# Run with volume for persistence
docker run -d -v mydata:/app/data myapp:1.0
```

### 3. Manage Container
```bash
# View logs
docker logs myapp
docker logs -f myapp  # Follow logs

# Execute command
docker exec -it myapp /bin/bash

# Stop container
docker stop myapp

# Start container
docker start myapp

# Remove container
docker rm myapp
```

### 4. Share Image
```bash
# Tag for Docker Hub
docker tag myapp:1.0 dockerhubusername/myapp:1.0

# Login to Docker Hub
docker login

# Push to registry
docker push dockerhubusername/myapp:1.0

# Others can pull
docker pull dockerhubusername/myapp:1.0
```

---

## Common Patterns

### Multi-Container Application (Docker Compose)

**docker-compose.yaml:**
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:8000"
    environment:
      - DATABASE_URL=postgresql://db/mydb
    depends_on:
      - db
    networks:
      - app-network

  db:
    image: postgres:14
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  postgres-data:

networks:
  app-network:
```

**Usage:**
```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop all services
docker-compose down
```

### Persistent Data with Volumes

```bash
# Create volume
docker volume create mydata

# Use volume in container
docker run -d -v mydata:/app/data myapp:1.0

# Data persists across container restarts
```

### Container Networking

```bash
# Create network
docker network create mynet

# Run containers on network
docker run -d --network mynet --name web nginx:1.23
docker run -d --network mynet --name app myapp:1.0

# Containers can communicate using service names
# e.g., from app container: curl http://web
```

---

## Troubleshooting Common Issues

### Container Won't Start
```bash
# Check logs
docker logs myapp

# Inspect container
docker inspect myapp

# Try running interactively
docker run -it myapp /bin/bash
```

### Port Already in Use
```bash
# Find which process uses port 8080
lsof -i :8080  # macOS/Linux
netstat -ano | findstr :8080  # Windows

# Use different port
docker run -d -p 9000:80 myapp:1.0
```

### Image Build Fails
```bash
# Build with verbose output
docker build -t myapp:1.0 . --no-cache

# Check Dockerfile syntax
# Common issues:
# - Base image doesn't exist (typo or wrong tag)
# - RUN commands fail (missing dependencies)
# - Working directory doesn't exist

# Use intermediate image for debugging
docker run -it <intermediate-image-id> /bin/bash
```

### Out of Disk Space
```bash
# Check usage
docker system df

# Clean up unused resources
docker system prune -a  # Remove all unused images
docker system prune --volumes  # Also remove unused volumes
```

### Container Running Out of Memory
```bash
# Monitor memory usage
docker stats myapp

# Set memory limit
docker run -d -m 512m myapp:1.0

# Check container resource consumption
docker stats --no-stream myapp
```

### Can't Connect to Container Service
```bash
# Verify port mapping
docker port myapp

# Check if service is listening
docker exec myapp netstat -tlnp

# Test from host
curl localhost:8080

# Check container IP
docker inspect --format='{{.NetworkSettings.IPAddress}}' myapp
```

### Permission Denied Issues
```bash
# Run as root
docker run -u root myapp:1.0

# Or add user to docker group (Linux)
sudo usermod -aG docker $USER
newgrp docker

# Create non-root user in image
docker run myapp:1.0 whoami  # Should not be root for security
```

---

## Performance Optimization

### Reduce Image Size
```dockerfile
# Use Alpine base image
FROM python:3.9-alpine

# Multi-stage build
FROM python:3.9 AS builder
# Build steps
FROM python:3.9-slim
# Copy only necessary files
COPY --from=builder /app /app

# Clean up
RUN apt-get purge && \
    rm -rf /var/lib/apt/lists/*
```

### Optimize Layers
```dockerfile
# Bad: Creates multiple layers
RUN apt-get update
RUN apt-get install -y package1
RUN apt-get install -y package2

# Good: Single layer
RUN apt-get update && \
    apt-get install -y \
    package1 \
    package2
```

### Use .dockerignore
```
.git
node_modules
.env
*.log
.DS_Store
```

### Enable BuildKit
```bash
# BuildKit is faster and more efficient
export DOCKER_BUILDKIT=1
docker build -t myapp:1.0 .
```

---

## Security Best Practices

### 1. Run as Non-Root User
```dockerfile
RUN useradd -m appuser
USER appuser
```

### 2. Don't Store Secrets in Images
```dockerfile
# Bad: Secret in layer
ENV DATABASE_PASSWORD=secret123

# Good: Pass as build arg or runtime env
ARG DATABASE_PASSWORD
ENV DATABASE_PASSWORD=${DATABASE_PASSWORD}
```

### 3. Scan Images for Vulnerabilities
```bash
# Using Docker Scout
docker scout cves myapp:1.0

# Using Trivy
trivy image myapp:1.0
```

### 4. Use Specific Base Image Tags
```dockerfile
# Avoid
FROM python:latest

# Use
FROM python:3.9.18-slim-bullseye
```

### 5. Keep Images Updated
```bash
# Rebuild images regularly to get security patches
docker build --no-cache -t myapp:1.0 .
```

---

## Common Docker Commands Quick Reference

| Task | Command |
|------|---------|
| Build image | `docker build -t name:tag .` |
| Run container | `docker run -d -p 8080:80 name:tag` |
| View containers | `docker ps -a` |
| View logs | `docker logs -f container_name` |
| Execute command | `docker exec -it container_name /bin/bash` |
| Stop container | `docker stop container_name` |
| Remove container | `docker rm container_name` |
| View images | `docker images` |
| Pull image | `docker pull name:tag` |
| Push image | `docker push username/name:tag` |
| Tag image | `docker tag source:tag username/target:tag` |
| Create network | `docker network create name` |
| Create volume | `docker volume create name` |
| View stats | `docker stats` |
| Docker Compose up | `docker-compose up -d` |
| Docker Compose down | `docker-compose down` |

---

## Environment Variables Management

### Method 1: Environment File
```bash
# Create .env file
DATABASE_URL=postgresql://localhost/mydb
NODE_ENV=production

# Use in docker-compose.yaml
services:
  app:
    env_file: .env

# Or from command line
docker run --env-file .env myapp:1.0
```

### Method 2: Direct Environment Variables
```bash
docker run -e VAR1=value1 -e VAR2=value2 myapp:1.0
```

### Method 3: In Dockerfile
```dockerfile
ENV NODE_ENV=production
ENV DEBUG=false
```

---

## Logging Best Practices

### 1. Log to STDOUT/STDERR
```dockerfile
# Good: Logs go to container stdout
CMD ["python", "-u", "app.py"]

# Avoid: Logging to file inside container
# Logs are lost when container is removed
```

### 2. View Logs Effectively
```bash
# Last 100 lines
docker logs --tail 100 myapp

# Follow logs
docker logs -f myapp

# With timestamps
docker logs -t myapp

# Since specific time
docker logs --since 2025-02-01 myapp
```

### 3. Log Drivers
```bash
# JSON file (default)
docker run -d \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  myapp:1.0
```

---

## Health Checks

### Dockerfile Healthcheck
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

### Check Health Status
```bash
docker ps --format "table {{.Names}}\t{{.Status}}"
```

---

## Container Networking Details

### Network Types
- **bridge**: Default, isolated network (host machine can't access containers directly)
- **host**: Container shares host network (faster, but less isolation)
- **overlay**: For swarm mode (multi-host)
- **none**: No network
- **macvlan**: Assign MAC address to container

### Example with Custom Network
```bash
# Create network
docker network create myapp-network

# Run containers on network
docker run -d --network myapp-network --name web nginx:1.23
docker run -d --network myapp-network --name api myapp:1.0

# Containers can communicate
docker exec api curl http://web
```

---

## Data Persistence Strategies

### Named Volumes (Recommended)
```bash
docker volume create mydata
docker run -d -v mydata:/app/data myapp:1.0
```

### Bind Mounts
```bash
docker run -d -v /host/path:/container/path myapp:1.0
```

### Tmpfs (Temporary, in-memory)
```bash
docker run -d --tmpfs /app/temp myapp:1.0
```

---

## Resource Management

### CPU Limits
```bash
# Single CPU
docker run -d --cpus 1 myapp:1.0

# Half CPU
docker run -d --cpus 0.5 myapp:1.0

# Specific CPUs (0, 1)
docker run -d --cpuset-cpus 0,1 myapp:1.0
```

### Memory Limits
```bash
# 512MB limit
docker run -d -m 512m myapp:1.0

# 512MB with 256MB swap
docker run -d -m 512m --memory-swap 768m myapp:1.0

# Monitor memory
docker stats myapp
```

---

## Useful Tips & Tricks

### View All Containers (Including Stopped)
```bash
docker ps -a
```

### Find Containers by Name Pattern
```bash
docker ps -a --filter "name=web"
```

### Stop All Running Containers
```bash
docker stop $(docker ps -q)
```

### Remove All Stopped Containers
```bash
docker rm $(docker ps -a -q)
```

### View Detailed Container Information
```bash
docker inspect myapp
docker inspect --format='{{.Config.Cmd}}' myapp
```

### Access Container Shell Safely
```bash
# Alpine: Use sh
docker exec -it myapp sh

# Debian/Ubuntu: Use bash
docker exec -it myapp bash
```

### Monitor Container in Real-Time
```bash
docker stats myapp
```

### Get Container IP Address
```bash
docker inspect --format='{{.NetworkSettings.IPAddress}}' myapp
```

### Copy Files Between Host and Container
```bash
# From container to host
docker cp myapp:/app/file.txt ./file.txt

# From host to container
docker cp ./file.txt myapp:/app/
```

### Create New Image from Container
```bash
docker commit myapp my-custom-image:1.0
```

---

## Docker Stats Output Explanation

```
CONTAINER ID   NAME       CPU %    MEM USAGE / LIMIT     MEM %    NET I/O
a1b2c3d4e5f6   myapp      2.50%    128MiB / 512MiB      25.00%   1.5MB / 2.3MB
```

- **CPU %**: CPU usage percentage
- **MEM USAGE / LIMIT**: Memory used / Memory limit
- **MEM %**: Memory usage percentage
- **NET I/O**: Network input / output

---

## Common Error Messages & Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| `Bind for 0.0.0.0:8080 failed: port is already allocated` | Port in use | Change port: `-p 9000:80` |
| `no such image` | Image doesn't exist locally | Pull image: `docker pull name:tag` |
| `Container exited with code 1` | Application error | Check logs: `docker logs container` |
| `Cannot connect to container` | Service not listening | Check: `docker exec container netstat -tlnp` |
| `permission denied` | User doesn't have docker access | Add user to group or use sudo |
| `OCI runtime error` | Container can't start | Check: `docker logs` and `docker inspect` |

---

## Resources & Learning

- **Official Docs**: https://docs.docker.com/
- **Docker Hub**: https://hub.docker.com/ (repository of images)
- **Interactive Tutorial**: https://labs.play-with-docker.com/
- **Awesome Docker**: https://github.com/veggiemonk/awesome-docker
- **Docker Best Practices**: https://docs.docker.com/develop/dev-best-practices/

---

## Final Tips

1. **Always read logs** when something goes wrong: `docker logs -f container`
2. **Use named volumes** for persistent data, not bind mounts
3. **Run containers with limits** to prevent resource exhaustion
4. **Don't run as root** in containers for security
5. **Use Docker Compose** for multi-container applications
6. **Tag images properly** with versions, not just `latest`
7. **Clean up regularly** to save disk space: `docker system prune`
8. **Monitor with docker stats** to optimize resource allocation
9. **Separate concerns** - one service per container
10. **Keep images small** - use Alpine base images when possible

