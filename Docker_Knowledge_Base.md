# Docker Knowledge Base

## Table of Contents
1. [Docker Fundamentals](#docker-fundamentals)
2. [Core Concepts](#core-concepts)
3. [Dockerfile Instructions](#dockerfile-instructions)
4. [Docker CLI Commands](#docker-cli-commands)
5. [Docker Compose](#docker-compose)
6. [Best Practices](#best-practices)
7. [Docker Architecture](#docker-architecture)
8. [Docker Commands](#docker-commands-reference---complete-list)
9. [containerd](#containerd--dockers-container-runtime)


## Docker Fundamentals

### What is Docker?

Docker is a containerization platform that packages applications along with all their dependencies (libraries, dependencies, configurations) into a standardized unit called a **container**. Containers are lightweight, portable, and ensure consistency across different environments—development, testing, and production.

### Key Benefits

- **Portability**: Containers run identically across different systems
- **Efficiency**: Uses fewer resources compared to virtual machines
- **Scalability**: Easy to scale applications horizontally
- **Isolation**: Each container is isolated from others, preventing dependency conflicts
- **Consistency**: Eliminates "works on my machine" problems
- **DevOps Integration**: Streamlines CI/CD pipelines and deployment workflows

---

## Core Concepts

### 1. Docker Container

A **Docker container** is a lightweight, standalone executable package that contains everything needed to run an application—code, runtime, system tools, libraries, and settings. It's essentially an instance of a Docker image.

**Key Characteristics:**
- Isolated from other containers
- Can be started, stopped, and removed
- Has its own filesystem, network interfaces, and process tree
- Ephemeral by nature (data is lost when container stops unless persisted via volumes)

### 2. Docker Image

A **Docker image** is a read-only blueprint or template used to create containers. It contains the application code, runtime environment, system libraries, and all dependencies. Images are built from a set of instructions defined in a Dockerfile.

**Image Layers:**
- Each instruction in a Dockerfile creates a new layer
- Layers are stacked on top of each other
- Each layer is cached, improving build efficiency
- Final image is the combination of all layers

**Image Registry:**
Images are stored in registries, the most popular being:
- **Docker Hub**: Official public registry (default)
- **ECR (AWS Elastic Container Registry)**: AWS's managed registry
- **GCR (Google Container Registry)**: Google's managed registry
- **Private Registries**: JFrog Artifactory, Harbor, or self-hosted registries

### 3. Docker Registry / Docker Hub

A **Docker registry** is a centralized repository where Docker images are stored and distributed. Docker Hub is the default public registry provided by Docker Inc.

**Common Registry Operations:**
- Pull images from registry
- Push custom images to registry
- Search for images
- Manage image versions and tags

### 4. Port Binding

**Port binding** connects a port on the host machine to a port inside the container, enabling external access to containerized services.

**Why Port Binding is Needed:**
- Containers have isolated network namespaces
- Applications running in containers listen on internal ports
- To access the service from outside the container, ports must be explicitly bound
- Without port binding, localhost:port will not work

**Example Scenario:**
- Nginx container runs on port 80 internally
- Host machine port 9000 is bound to container port 80
- Accessing localhost:9000 on the host routes traffic to port 80 inside the container

---

## Dockerfile Instructions

A **Dockerfile** is a text file containing instructions to build a Docker image. Each instruction creates a new layer in the image.

### Basic Dockerfile Structure

```dockerfile
# Base image
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Copy files from host to container
COPY . /app

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Expose port
EXPOSE 5000

# Set environment variables
ENV APP_ENV=production

# Set the user to run the container
USER appuser

# Default command to run when container starts
CMD ["python", "app.py"]
```

### Dockerfile Instructions Reference

| Instruction | Purpose | Example |
|---|---|---|
| **FROM** | Specifies the base image | `FROM python:3.9` |
| **WORKDIR** | Sets the working directory for subsequent instructions | `WORKDIR /app` |
| **COPY** | Copies files/directories from host to container | `COPY . /app` |
| **ADD** | Similar to COPY but with additional features (auto-extract archives) | `ADD app.tar.gz /app` |
| **RUN** | Executes commands during image build | `RUN apt-get update && apt-get install -y git` |
| **EXPOSE** | Declares ports the container listens on (documentation only) | `EXPOSE 8080` |
| **ENV** | Sets environment variables in the image | `ENV DATABASE_URL=postgres://localhost` |
| **ARG** | Defines build-time arguments (not available at runtime) | `ARG BUILD_VERSION=1.0` |
| **USER** | Specifies the user to run the container | `USER appuser` |
| **CMD** | Provides default command to execute when container starts | `CMD ["python", "app.py"]` |
| **ENTRYPOINT** | Configures the container to run as an executable | `ENTRYPOINT ["java", "-jar"]` |
| **VOLUME** | Creates a mount point for persistent data | `VOLUME ["/data"]` |
| **LABEL** | Adds metadata to the image | `LABEL version="1.0" maintainer="john@example.com"` |

### Dockerfile Best Practices

1. **Use Specific Base Image Versions**: Avoid `latest` tag to prevent unexpected breaking changes
   ```dockerfile
   # Bad
   FROM python:latest
   
   # Good
   FROM python:3.9-slim
   ```

2. **Minimize Layer Count**: Combine RUN commands with && to reduce layers
   ```dockerfile
   # Avoid
   RUN apt-get update
   RUN apt-get install -y curl
   RUN apt-get install -y git
   
   # Better
   RUN apt-get update && apt-get install -y \
       curl \
       git
   ```

3. **Use .dockerignore**: Exclude unnecessary files to reduce build context
   ```
   .git
   node_modules
   .env
   *.log
   ```

4. **Run as Non-Root User**: Improves security
   ```dockerfile
   RUN useradd -m appuser
   USER appuser
   ```

5. **Order Instructions**: Place frequently changing instructions last to leverage caching
   ```dockerfile
   FROM python:3.9
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install -r requirements.txt
   COPY . .  # Place this after RUN to benefit from cache
   CMD ["python", "app.py"]
   ```

6. **Multi-Stage Builds**: Reduce final image size by using multiple FROM statements
   ```dockerfile
   # Stage 1: Build
   FROM python:3.9 as builder
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install --user -r requirements.txt
   
   # Stage 2: Runtime
   FROM python:3.9-slim
   WORKDIR /app
   COPY --from=builder /root/.local .local
   COPY . .
   CMD ["python", "app.py"]
   ```

7. **Use Alpine Images**: Smaller base images save bandwidth and storage
   ```dockerfile
   FROM python:3.9-alpine  # ~50MB vs 900MB for full image
   ```

---

## Docker CLI Commands

### Image Management Commands

| Command | Description | Usage |
|---|---|---|
| **docker images** | Lists all local images | `docker images` |
| **docker pull** | Downloads image from registry | `docker pull nginx:1.23` |
| **docker push** | Uploads image to registry | `docker push myrepo/myimage:1.0` |
| **docker build** | Builds image from Dockerfile | `docker build -t myapp:1.0 .` |
| **docker tag** | Creates alias for an image | `docker tag myapp:1.0 myrepo/myapp:1.0` |
| **docker rmi** | Removes local image | `docker rmi myapp:1.0` |
| **docker search** | Searches Docker Hub for images | `docker search nginx` |
| **docker inspect** | Returns detailed image information in JSON format | `docker inspect myimage:1.0` |
| **docker history** | Shows image layer history | `docker history myimage:1.0` |
| **docker save** | Saves image to tar archive | `docker save myimage:1.0 > myimage.tar` |
| **docker load** | Loads image from tar archive | `docker load < myimage.tar` |

#### Image Commands Examples

```bash
# Pull specific version of nginx
docker pull nginx:1.23

# Pull latest version (not recommended for production)
docker pull nginx

# Build image with tag
docker build -t nodeapp:1.0 .

# Tag image for Docker Hub
docker tag nodeapp:1.0 dockerhubusername/nodeapp:1.0

# Push to Docker Hub
docker push dockerhubusername/nodeapp:1.0

# View image history and layers
docker history myapp:1.0

# Inspect detailed image metadata
docker inspect myapp:1.0
```

---

### Container Management Commands

| Command | Description | Usage |
|---|---|---|
| **docker run** | Creates and starts a new container | `docker run -d -p 8080:80 nginx:1.23` |
| **docker create** | Creates container without starting it | `docker create --name web nginx:1.23` |
| **docker start** | Starts a stopped container | `docker start mycontainer` |
| **docker stop** | Stops a running container gracefully | `docker stop mycontainer` |
| **docker kill** | Forcefully terminates a container | `docker kill mycontainer` |
| **docker restart** | Stops and restarts a container | `docker restart mycontainer` |
| **docker rm** | Removes a container | `docker rm mycontainer` |
| **docker ps** | Lists running containers | `docker ps` |
| **docker ps -a** | Lists all containers (running and stopped) | `docker ps -a` |
| **docker logs** | Views container output/logs | `docker logs mycontainer` |
| **docker exec** | Executes command in running container | `docker exec -it mycontainer /bin/bash` |
| **docker attach** | Attaches terminal to running container | `docker attach mycontainer` |
| **docker rename** | Renames a container | `docker rename oldname newname` |
| **docker pause** | Pauses all processes in a container | `docker pause mycontainer` |
| **docker unpause** | Resumes paused container | `docker unpause mycontainer` |
| **docker stats** | Displays real-time container resource usage | `docker stats` |
| **docker top** | Shows processes running in container | `docker top mycontainer` |
| **docker commit** | Creates new image from container changes | `docker commit mycontainer newimage:1.0` |
| **docker diff** | Shows filesystem changes in container | `docker diff mycontainer` |
| **docker export** | Exports container filesystem as tar | `docker export mycontainer > export.tar` |
| **docker cp** | Copies files between host and container | `docker cp mycontainer:/path/file ./local/` |
| **docker port** | Lists port mappings for container | `docker port mycontainer` |

#### Container Commands Examples

```bash
# Run nginx in background with port binding
docker run -d -p 9000:80 nginx:1.23

# Run container interactively with terminal
docker run -it ubuntu /bin/bash

# Run container with name
docker run --name myapp -d -p 3000:3000 node-app:1.0

# View logs
docker logs myapp

# View logs in real-time
docker logs -f myapp

# Execute command in running container
docker exec -it myapp /bin/bash

# Create file in running container
docker exec myapp touch /tmp/testfile

# List all containers
docker ps -a

# Stop all running containers
docker stop $(docker ps -q)

# Remove stopped container
docker rm myapp

# Display container resource usage
docker stats myapp

# Copy file from container
docker cp myapp:/app/config.json ./config.json

# Copy file to container
docker cp ./config.json myapp:/app/
```

---

### Network Commands

| Command | Description | Usage |
|---|---|---|
| **docker network create** | Creates a new network | `docker network create mynetwork` |
| **docker network ls** | Lists all networks | `docker network ls` |
| **docker network inspect** | Shows detailed network information | `docker network inspect mynetwork` |
| **docker network connect** | Connects running container to network | `docker network connect mynetwork mycontainer` |
| **docker network disconnect** | Disconnects container from network | `docker network disconnect mynetwork mycontainer` |
| **docker network rm** | Removes a network | `docker network rm mynetwork` |
| **docker network prune** | Removes unused networks | `docker network prune` |

#### Network Commands Examples

```bash
# Create bridge network
docker network create -d bridge my-network

# Connect existing container to network
docker network connect my-network mycontainer

# Run container on specific network
docker run -d --name web1 --network my-network nginx:1.23

# Inspect network to see connected containers
docker network inspect my-network

# Disconnect container from network
docker network disconnect my-network mycontainer

# Remove network (ensure no containers are connected)
docker network rm my-network
```

---

### Volume Management Commands

| Command | Description | Usage |
|---|---|---|
| **docker volume create** | Creates a named volume | `docker volume create myvolume` |
| **docker volume ls** | Lists all volumes | `docker volume ls` |
| **docker volume inspect** | Shows detailed volume information | `docker volume inspect myvolume` |
| **docker volume rm** | Removes a volume | `docker volume rm myvolume` |
| **docker volume prune** | Removes unused volumes | `docker volume prune` |

#### Volume Commands Examples

```bash
# Create volume
docker volume create data-volume

# Mount volume to container
docker run -d -v data-volume:/data nginx:1.23

# Mount with bind-mount (host directory)
docker run -d -v /host/path:/container/path nginx:1.23

# Inspect volume details
docker volume inspect data-volume

# List all volumes
docker volume ls

# Remove volume
docker volume rm data-volume
```

---

### Docker System Commands

| Command | Description | Usage |
|---|---|---|
| **docker version** | Shows Docker version information | `docker version` |
| **docker info** | Displays system-wide information | `docker info` |
| **docker login** | Authenticates to Docker registry | `docker login` |
| **docker logout** | Logs out from registry | `docker logout` |
| **docker system df** | Shows Docker disk usage | `docker system df` |
| **docker system prune** | Removes unused Docker objects | `docker system prune` |
| **docker events** | Streams Docker events in real-time | `docker events` |

---

### Port Binding Examples

```bash
# Simple port binding (host:container)
docker run -d -p 8080:80 nginx:1.23
# Host port 8080 → Container port 80

# Bind to specific host IP
docker run -d -p 192.168.1.100:8080:80 nginx:1.23

# Bind multiple ports
docker run -d -p 8080:80 -p 8443:443 nginx:1.23

# Bind to random host port
docker run -d -p 80 nginx:1.23
# Docker assigns random port from ephemeral range

# View port mappings
docker port mycontainer
```

---

## Docker Compose

Docker Compose is a tool for defining and running multi-container Docker applications using a YAML file.

### Key Concepts

- **Not a replacement for Docker**: You still write Dockerfiles and docker-compose.yaml
- **Orchestrates containers**: Manages multiple containers, networks, and volumes
- **Execution order**: Can specify dependencies between services
- **Network isolation**: Services can communicate using service names as hostnames

### docker-compose.yaml Structure

```yaml
version: '3.8'

services:
  # Frontend service
  frontend:
    image: nginx:latest
    ports:
      - "80:80"
    depends_on:
      - web1
      - web2
    networks:
      - frontend-network

  # Web server 1
  web1:
    build: ./app
    ports:
      - "3001:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://db:5432/mydb
    depends_on:
      - db
    networks:
      - backend-network

  # Web server 2
  web2:
    build: ./app
    ports:
      - "3002:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://db:5432/mydb
    depends_on:
      - db
    networks:
      - backend-network

  # Redis cache
  cache:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    networks:
      - backend-network

  # PostgreSQL database
  db:
    image: postgres:14
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - backend-network

volumes:
  postgres-data:

networks:
  frontend-network:
    driver: bridge
  backend-network:
    driver: bridge
```

### Docker Compose Commands

| Command | Description | Usage |
|---|---|---|
| **docker-compose up** | Creates and starts all services | `docker-compose up` |
| **docker-compose up -d** | Starts services in background | `docker-compose up -d` |
| **docker-compose down** | Stops and removes all containers | `docker-compose down` |
| **docker-compose build** | Builds images for services | `docker-compose build` |
| **docker-compose start** | Starts stopped services | `docker-compose start` |
| **docker-compose stop** | Stops running services | `docker-compose stop` |
| **docker-compose restart** | Restarts services | `docker-compose restart` |
| **docker-compose logs** | Views service logs | `docker-compose logs` |
| **docker-compose logs -f** | Follows logs in real-time | `docker-compose logs -f` |
| **docker-compose ps** | Lists services and their status | `docker-compose ps` |
| **docker-compose exec** | Executes command in service | `docker-compose exec web /bin/bash` |
| **docker-compose scale** | Scales service to multiple instances | `docker-compose scale web=3` |
| **docker-compose rm** | Removes stopped services | `docker-compose rm` |
| **docker-compose pull** | Pulls service images | `docker-compose pull` |

#### Docker Compose Examples

```bash
# Start all services
docker-compose up -d

# View service status
docker-compose ps

# View logs
docker-compose logs web1

# Follow logs in real-time
docker-compose logs -f web1

# Execute command in service
docker-compose exec web1 /bin/bash

# Stop services
docker-compose stop

# Stop and remove containers
docker-compose down

# Remove volumes as well
docker-compose down -v

# Scale a service
docker-compose up -d --scale web1=3
```

---

## Best Practices

### Image Building Best Practices

1. **Use Minimal Base Images**
   - Alpine images (~5MB) instead of full OS images (~900MB)
   - Reduces storage, bandwidth, and attack surface

2. **Order Dockerfile Instructions Strategically**
   - Place frequently changing instructions last
   - Leverage Docker layer caching for faster builds

3. **Exclude Unnecessary Files**
   - Create .dockerignore file
   - Reduce build context size

4. **Pin Dependency Versions**
   - Prevents unexpected breaking changes
   - Ensures reproducible builds

5. **Use Environment Variables**
   - Make containers flexible without rebuilding
   - Keep configuration separate from code

6. **Implement Security Practices**
   - Run containers as non-root users
   - Avoid using `latest` tags
   - Scan images for vulnerabilities

### Container Runtime Best Practices

1. **Resource Limits**
   ```bash
   docker run -d -m 512m --cpus 1 nginx:1.23
   # Memory limit: 512MB
   # CPU limit: 1 core
   ```

2. **Health Checks**
   ```dockerfile
   HEALTHCHECK --interval=30s --timeout=3s \
     CMD curl -f http://localhost/ || exit 1
   ```

3. **Logging**
   - Ensure applications log to STDOUT/STDERR
   - Avoid writing logs to files inside containers

4. **Graceful Shutdown**
   - Handle SIGTERM signal for clean shutdown
   - Give containers time to clean up before SIGKILL

5. **Volume Usage**
   - Use volumes for persistent data
   - Avoid storing data in container filesystem

### Docker Compose Best Practices

1. **Understand Project Dependencies**
   - Plan service architecture before writing compose file
   - Use depends_on to specify startup order

2. **Use Version-Pinned Tags**
   - Avoid `latest` tag in production
   - Pin specific versions for reproducibility

3. **Environment Management**
   - Use .env files for configuration
   - Keep sensitive data out of compose files

4. **Resource Constraints**
   ```yaml
   services:
     web:
       image: myapp:1.0
       deploy:
         resources:
           limits:
             cpus: '1'
             memory: 512M
   ```

5. **Networking**
   - Use custom networks instead of default bridge
   - Services communicate using service names

---

## Docker Architecture

### How Docker Works

```
┌─────────────────────────────────────────┐
│         Docker Client (CLI)             │
│  docker run, docker build, etc.         │
└──────────────┬──────────────────────────┘
               │ (Docker API)
               │
┌──────────────▼──────────────────────────┐
│       Docker Daemon (dockerd)           │
│  - Manages images, containers, networks │
│  - Manages volumes and storage          │
│  - Handles networking and isolation     │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│    Container Runtime (containerd)       │
│  - Creates and manages containers       │
│  - Manages namespaces and cgroups      │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│      Operating System (Linux)           │
│  - Namespaces: PID, NET, IPC, MNT, UTS │
│  - Control Groups (cgroups): CPU, MEM  │
│  - Union Filesystem: Overlay2, AUFS     │
└─────────────────────────────────────────┘
```

### Container Isolation Technologies

1. **Namespaces**: Provide process isolation
   - PID Namespace: Process IDs
   - Network Namespace: Network interfaces
   - Mount Namespace: Filesystems
   - IPC Namespace: Inter-process communication
   - UTS Namespace: Hostname and domain name
   - User Namespace: User IDs

2. **Control Groups (cgroups)**: Manage resource limits
   - CPU usage
   - Memory limits
   - I/O limits
   - Device access

3. **Union Filesystem**: Efficient layered storage
   - Multiple read-only layers
   - Single writable layer per container
   - Reduces storage and improves performance

---

## Quick Reference Cheat Sheet

### Most Used Commands

```bash
# Build image
docker build -t myapp:1.0 .

# Run container
docker run -d -p 8080:80 --name myapp myapp:1.0

# View containers
docker ps -a

# View logs
docker logs myapp

# Stop container
docker stop myapp

# Remove container
docker rm myapp

# Push to registry
docker tag myapp:1.0 dockerhubusername/myapp:1.0
docker push dockerhubusername/myapp:1.0

# Pull from registry
docker pull nginx:1.23

# Docker Compose
docker-compose up -d
docker-compose down
docker-compose logs -f
```

---

## Resources

- Official Docker Documentation: https://docs.docker.com/
- Docker Hub: https://hub.docker.com/
- Awesome Compose Repository: https://github.com/docker/awesome-compose (Examples of docker-compose files)
- Docker Community Guidelines: https://www.docker.com/community

---

## Conclusion

Docker revolutionizes application deployment by providing consistent, reproducible, and isolated environments. Mastering Docker fundamentals—containers, images, Dockerfiles, and Docker Compose—enables efficient DevOps practices and scalable application architectures. Understanding the underlying technologies (namespaces, cgroups, union filesystems) provides deeper insights into container behavior and troubleshooting.

Recommended learning path:
1. Understand Docker fundamentals and core concepts
2. Master Dockerfile writing and best practices
3. Learn container management and networking
4. Advance to Docker Compose for multi-container applications
5. Explore orchestration tools (Kubernetes) for production deployments


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

# Docker Commands Reference - Complete List

## Docker CLI Command Syntax

```
docker [OPTIONS] COMMAND [ARG...]
```

---

## Image Management Commands

### docker build
Builds an image from a Dockerfile.

```bash
docker build [OPTIONS] PATH | URL | -
```

**Common Options:**
- `-t, --tag`: Name and optional tag in 'name:tag' format
- `-f, --file`: Name of the Dockerfile
- `--build-arg`: Set build-time variables
- `--cache-from`: Images to consider for cache sources
- `--no-cache`: Do not use cache when building image

**Examples:**
```bash
docker build -t myapp:1.0 .
docker build -t myapp:1.0 -f Dockerfile.prod .
docker build --build-arg BUILD_DATE=2025-02-01 -t myapp:1.0 .
docker build --no-cache -t myapp:1.0 .
```

---

### docker images
Lists all local images.

```bash
docker images [OPTIONS] [REPOSITORY[:TAG]]
```

**Common Options:**
- `-a, --all`: Show all images (including intermediate images)
- `-q, --quiet`: Only show numeric IDs
- `--filter`: Filter output based on conditions
- `--format`: Pretty-print images using a Go template
- `--digests`: Show digests (SHA256) of images

**Examples:**
```bash
docker images
docker images -a
docker images -q
docker images ubuntu
docker images --filter "dangling=true"
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

---

### docker pull
Pulls an image from a registry.

```bash
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

**Common Options:**
- `-a, --all-tags`: Download all tagged images from repository
- `-q, --quiet`: Suppress verbose output
- `--disable-content-trust`: Skip image verification

**Examples:**
```bash
docker pull nginx:1.23
docker pull nginx  # Pulls latest
docker pull ubuntu:20.04
docker pull gcr.io/google-containers/nginx:latest
docker pull -a nginx  # Pull all tags
```

---

### docker push
Pushes an image to a registry.

```bash
docker push [OPTIONS] NAME[:TAG]
```

**Common Options:**
- `-a, --all-tags`: Push all tags of an image
- `-q, --quiet`: Suppress verbose output
- `--disable-content-trust`: Skip image signing

**Examples:**
```bash
docker push myrepo/myapp:1.0
docker push dockerhubusername/myapp:1.0
docker push localhost:5000/myapp:1.0
docker push -a myrepo/myapp
```

---

### docker tag
Creates a tag for an image.

```bash
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

**Examples:**
```bash
docker tag myapp:1.0 dockerhubusername/myapp:1.0
docker tag nginx:1.23 localhost:5000/nginx:1.23
docker tag 5d12a3e /nginx:latest
```

---

### docker rmi
Removes one or more images.

```bash
docker rmi [OPTIONS] IMAGE [IMAGE...]
```

**Common Options:**
- `-f, --force`: Force removal of image
- `--no-prune`: Do not delete untagged parent images

**Examples:**
```bash
docker rmi myapp:1.0
docker rmi ubuntu:20.04 python:3.9
docker rmi -f myapp:1.0
docker rmi $(docker images -q)  # Remove all images
```

---

### docker search
Searches Docker Hub for images.

```bash
docker search [OPTIONS] TERM
```

**Common Options:**
- `--limit`: Maximum number of search results
- `--filter`: Filter output

**Examples:**
```bash
docker search nginx
docker search --limit 10 ubuntu
docker search --filter "is-official=true" python
```

---

### docker inspect
Returns low-level information about Docker objects.

```bash
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
```

**Common Options:**
- `-f, --format`: Format the output using a Go template
- `--type`: Return JSON for specified type (image, container, etc.)

**Examples:**
```bash
docker inspect myapp:1.0
docker inspect --format='{{.Config.ExposedPorts}}' myimage:1.0
docker inspect --type=image myapp:1.0
docker inspect --format='{{.Architecture}}' ubuntu:20.04
```

---

### docker history
Shows the history of an image (layers).

```bash
docker history [OPTIONS] IMAGE
```

**Common Options:**
- `--no-trunc`: Do not truncate output
- `-q, --quiet`: Only show numeric IDs
- `--format`: Pretty-print using a Go template

**Examples:**
```bash
docker history myapp:1.0
docker history --no-trunc myapp:1.0
docker history -q myapp:1.0
```

---

### docker save
Saves an image to a tar archive.

```bash
docker save [OPTIONS] IMAGE [IMAGE...]
```

**Common Options:**
- `-o, --output`: Write to a file instead of STDOUT

**Examples:**
```bash
docker save myapp:1.0 > myapp.tar
docker save -o myapp.tar myapp:1.0
docker save ubuntu:20.04 python:3.9 | gzip > images.tar.gz
```

---

### docker load
Loads an image from a tar archive.

```bash
docker load [OPTIONS]
```

**Common Options:**
- `-i, --input`: Read from tar archive file
- `-q, --quiet`: Suppress verbose output

**Examples:**
```bash
docker load < myapp.tar
docker load -i myapp.tar
docker load -i images.tar.gz
```

---

## Container Management Commands

### docker run
Creates and runs a new container.

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

**Common Options:**
- `-d, --detach`: Run container in background
- `-i, --interactive`: Keep STDIN open even if not attached
- `-t, --tty`: Allocate a pseudo-TTY
- `-p, --publish`: Publish container port(s) to the host
- `-e, --env`: Set environment variables
- `--env-file`: Read in a file of environment variables
- `-v, --volume`: Bind mount a volume
- `--name`: Assign a name to container
- `--hostname`: Container host name
- `-m, --memory`: Memory limit
- `--cpus`: Number of CPUs to use
- `--restart`: Restart policy (no, always, on-failure, unless-stopped)
- `--network`: Connect to a network
- `-u, --user`: Username or UID to run the container
- `--privileged`: Give extended privileges
- `--rm`: Automatically remove container when it exits
- `-w, --workdir`: Working directory inside container
- `--link`: Add link to another container
- `-l, --label`: Set metadata labels

**Examples:**
```bash
# Basic run
docker run nginx:1.23

# Run in background (detached mode)
docker run -d nginx:1.23

# Run with interactive terminal
docker run -it ubuntu /bin/bash

# Run with port mapping
docker run -d -p 8080:80 nginx:1.23
docker run -d -p 8080:80 -p 8443:443 nginx:1.23

# Run with environment variables
docker run -d -e DATABASE_URL=postgres://localhost myapp:1.0
docker run -d --env-file .env myapp:1.0

# Run with volume
docker run -d -v myvolume:/data nginx:1.23
docker run -d -v /host/path:/container/path nginx:1.23

# Run with name
docker run --name web -d nginx:1.23

# Run with resource limits
docker run -d -m 512m --cpus 1 nginx:1.23

# Run with restart policy
docker run -d --restart unless-stopped nginx:1.23
docker run -d --restart on-failure:5 myapp:1.0

# Run with network
docker run -d --network mynetwork --name web nginx:1.23

# Run as specific user
docker run -d -u appuser nginx:1.23

# Remove container on exit
docker run --rm myapp:1.0

# Combined options
docker run -d \
  --name myapp \
  -p 3000:3000 \
  -e NODE_ENV=production \
  -v app-data:/data \
  --memory 512m \
  --cpus 1 \
  --restart unless-stopped \
  myapp:1.0
```

---

### docker create
Creates a new container (does not start it).

```bash
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

**Options:** Same as docker run

**Examples:**
```bash
docker create --name myapp nginx:1.23
docker create -p 8080:80 --name web nginx:1.23
```

---

### docker start
Starts one or more stopped containers.

```bash
docker start [OPTIONS] CONTAINER [CONTAINER...]
```

**Common Options:**
- `-a, --attach`: Attach STDOUT/STDERR and forward signals
- `-i, --interactive`: Attach container STDIN

**Examples:**
```bash
docker start myapp
docker start web1 web2
docker start -a myapp
```

---

### docker stop
Stops one or more running containers.

```bash
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```

**Common Options:**
- `-t, --time`: Seconds to wait before killing container (default 10)

**Examples:**
```bash
docker stop myapp
docker stop web1 web2
docker stop -t 30 myapp
docker stop $(docker ps -q)  # Stop all running containers
```

---

### docker kill
Forcefully kills one or more running containers.

```bash
docker kill [OPTIONS] CONTAINER [CONTAINER...]
```

**Common Options:**
- `-s, --signal`: Signal to send to container (default SIGKILL)

**Examples:**
```bash
docker kill myapp
docker kill -s SIGTERM myapp
docker kill web1 web2
```

---

### docker restart
Restarts one or more containers.

```bash
docker restart [OPTIONS] CONTAINER [CONTAINER...]
```

**Common Options:**
- `-t, --time`: Seconds to wait before killing container

**Examples:**
```bash
docker restart myapp
docker restart web1 web2
docker restart -t 30 myapp
```

---

### docker rm
Removes one or more containers.

```bash
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

**Common Options:**
- `-f, --force`: Force the removal
- `-v, --volumes`: Remove volumes associated with container
- `-l, --link`: Remove link
- `--name`: Interpret each argument as container name

**Examples:**
```bash
docker rm myapp
docker rm web1 web2
docker rm -f myapp  # Force remove even if running
docker rm -f -v myapp  # Remove container and volumes
docker rm $(docker ps -a -q)  # Remove all containers
```

---

### docker ps
Lists containers.

```bash
docker ps [OPTIONS]
```

**Common Options:**
- `-a, --all`: Show all containers (default only running)
- `-q, --quiet`: Only display numeric IDs
- `-n, --last`: Show last n containers
- `-s, --size`: Display total file sizes
- `-f, --filter`: Filter output
- `--format`: Pretty-print containers using Go template

**Examples:**
```bash
docker ps
docker ps -a
docker ps -a -q
docker ps -n 5
docker ps -a -s
docker ps -a --filter "status=running"
docker ps -a --filter "name=web"
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

---

### docker logs
Fetches logs of a container.

```bash
docker logs [OPTIONS] CONTAINER
```

**Common Options:**
- `-f, --follow`: Follow log output (like tail -f)
- `--tail`: Number of lines to show from end
- `-t, --timestamps`: Show timestamps
- `--since`: Show logs since timestamp
- `--until`: Show logs until timestamp

**Examples:**
```bash
docker logs myapp
docker logs -f myapp  # Follow logs
docker logs --tail 50 myapp  # Last 50 lines
docker logs -t myapp  # With timestamps
docker logs --since 2025-02-01T00:00:00 myapp
docker logs --tail 100 -f myapp
```

---

### docker exec
Executes a command in a running container.

```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

**Common Options:**
- `-d, --detach`: Detached mode (run command in background)
- `-i, --interactive`: Keep STDIN open
- `-t, --tty`: Allocate pseudo-TTY
- `-u, --user`: Username or UID to run command as
- `-w, --workdir`: Working directory to run command
- `-e, --env`: Set environment variables
- `--privileged`: Give extended privileges

**Examples:**
```bash
docker exec myapp ls -la
docker exec -it myapp /bin/bash
docker exec -it myapp /bin/sh
docker exec -d myapp touch /tmp/testfile
docker exec -u root myapp apt-get update
docker exec -it myapp python app.py
docker exec -w /app myapp npm start
```

---

### docker attach
Attaches to a running container.

```bash
docker attach [OPTIONS] CONTAINER
```

**Options:**
- `--sig-proxy`: Proxy all received signals to process (default true)

**Examples:**
```bash
docker attach myapp
docker attach --sig-proxy=false myapp
```

---

### docker commit
Creates a new image from a container's changes.

```bash
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

**Common Options:**
- `-a, --author`: Author name
- `-m, --message`: Commit message
- `-c, --change`: Dockerfile instruction to apply
- `-p, --pause`: Pause container during commit (default true)

**Examples:**
```bash
docker commit myapp newimage:1.0
docker commit -a "John Doe" -m "Added dependencies" myapp myapp:2.0
docker commit -c 'CMD ["python", "app.py"]' myapp myapp:1.0
```

---

### docker cp
Copies files/directories between host and container.

```bash
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
```

**Common Options:**
- `-L, --follow-link`: Always follow symbol links in SRC_PATH
- `-a, --archive`: Archive mode (copy all uid/gid info)

**Examples:**
```bash
# Copy FROM container TO host
docker cp myapp:/app/config.json ./config.json
docker cp myapp:/app ./local-app

# Copy FROM host TO container
docker cp ./config.json myapp:/app/
docker cp ./data myapp:/app/
```

---

### docker rename
Renames a container.

```bash
docker rename CONTAINER NEW_NAME
```

**Examples:**
```bash
docker rename myapp myapp-old
docker rename oldname newname
```

---

### docker pause
Pauses all processes in a container.

```bash
docker pause CONTAINER [CONTAINER...]
```

**Examples:**
```bash
docker pause myapp
docker pause web1 web2
```

---

### docker unpause
Unpauses all processes in a container.

```bash
docker unpause CONTAINER [CONTAINER...]
```

**Examples:**
```bash
docker unpause myapp
docker unpause web1 web2
```

---

### docker stats
Displays real-time container resource usage statistics.

```bash
docker stats [OPTIONS] [CONTAINER...]
```

**Common Options:**
- `--no-stream`: Disable streaming stats and only pull the first result
- `--format`: Pretty-print stats using Go template
- `-a, --all`: Show all containers (default only running)

**Examples:**
```bash
docker stats
docker stats myapp
docker stats web1 web2
docker stats --no-stream
docker stats --all
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

---

### docker top
Displays running processes in a container.

```bash
docker top CONTAINER [ps OPTIONS]
```

**Examples:**
```bash
docker top myapp
docker top myapp aux
docker top myapp -elf
```

---

### docker diff
Inspects changes to files or directories on a container's filesystem.

```bash
docker diff CONTAINER
```

**Examples:**
```bash
docker diff myapp
```

---

### docker export
Exports a container's filesystem as a tar archive.

```bash
docker export [OPTIONS] CONTAINER
```

**Common Options:**
- `-o, --output`: Write to file instead of STDOUT

**Examples:**
```bash
docker export myapp > myapp.tar
docker export -o myapp.tar myapp
```

---

### docker port
Lists port mappings for a container.

```bash
docker port CONTAINER [PRIVATE_PORT[/PROTO]]
```

**Examples:**
```bash
docker port myapp
docker port myapp 80
docker port myapp 8080/tcp
```

---

## Network Commands

### docker network create
Creates a new network.

```bash
docker network create [OPTIONS] NETWORK
```

**Common Options:**
- `-d, --driver`: Driver to manage network (bridge, host, overlay, macvlan, none)
- `--subnet`: Subnet for the network
- `--gateway`: Gateway for the master subnet
- `--ip-range`: Allocate container IP from a subnet
- `-o, --opt`: Set driver specific options
- `--label`: Set metadata on network

**Examples:**
```bash
docker network create mynetwork
docker network create -d bridge mybridge
docker network create -d overlay myoverlay
docker network create --subnet 172.20.0.0/16 mynetwork
docker network create --subnet 172.20.0.0/16 --gateway 172.20.0.1 mynetwork
```

---

### docker network ls
Lists all networks.

```bash
docker network ls [OPTIONS]
```

**Common Options:**
- `-q, --quiet`: Only display network IDs
- `-f, --filter`: Filter output
- `--format`: Pretty-print networks using Go template

**Examples:**
```bash
docker network ls
docker network ls -q
docker network ls --filter "driver=bridge"
```

---

### docker network inspect
Shows detailed network information.

```bash
docker network inspect [OPTIONS] NETWORK [NETWORK...]
```

**Common Options:**
- `-f, --format`: Pretty-print using Go template

**Examples:**
```bash
docker network inspect mynetwork
docker network inspect mynetwork --format='{{json .Containers}}'
docker network inspect network1 network2
```

---

### docker network connect
Connects a container to a network.

```bash
docker network connect [OPTIONS] NETWORK CONTAINER
```

**Common Options:**
- `--ip`: IPv4 address for the container on this network
- `--ip6`: IPv6 address for the container on this network
- `--alias`: Add network-scoped alias
- `--link`: Add link to another container

**Examples:**
```bash
docker network connect mynetwork myapp
docker network connect --ip 172.20.0.10 mynetwork myapp
docker network connect --alias web mynetwork myapp
```

---

### docker network disconnect
Disconnects a container from a network.

```bash
docker network disconnect [OPTIONS] NETWORK CONTAINER
```

**Common Options:**
- `-f, --force`: Force disconnect

**Examples:**
```bash
docker network disconnect mynetwork myapp
docker network disconnect -f mynetwork myapp
```

---

### docker network rm
Removes one or more networks.

```bash
docker network rm NETWORK [NETWORK...]
```

**Examples:**
```bash
docker network rm mynetwork
docker network rm network1 network2
```

---

### docker network prune
Removes all unused networks.

```bash
docker network prune [OPTIONS]
```

**Common Options:**
- `-f, --force`: Do not prompt for confirmation
- `--filter`: Provide filter values

**Examples:**
```bash
docker network prune
docker network prune -f
```

---

## Volume Commands

### docker volume create
Creates a new volume.

```bash
docker volume create [OPTIONS] [VOLUME_NAME]
```

**Common Options:**
- `-d, --driver`: Volume driver name
- `-o, --opt`: Set driver specific options
- `--label`: Set metadata labels

**Examples:**
```bash
docker volume create myvolume
docker volume create -d local myvolume
docker volume create --label env=prod myvolume
```

---

### docker volume ls
Lists volumes.

```bash
docker volume ls [OPTIONS]
```

**Common Options:**
- `-q, --quiet`: Only display volume names
- `-f, --filter`: Filter output
- `--format`: Pretty-print volumes using Go template

**Examples:**
```bash
docker volume ls
docker volume ls -q
docker volume ls --filter "driver=local"
```

---

### docker volume inspect
Displays detailed volume information.

```bash
docker volume inspect [OPTIONS] VOLUME [VOLUME...]
```

**Examples:**
```bash
docker volume inspect myvolume
docker volume inspect volume1 volume2
```

---

### docker volume rm
Removes one or more volumes.

```bash
docker volume rm [OPTIONS] VOLUME [VOLUME...]
```

**Common Options:**
- `-f, --force`: Force removal

**Examples:**
```bash
docker volume rm myvolume
docker volume rm volume1 volume2
docker volume rm -f myvolume
```

---

### docker volume prune
Removes all unused volumes.

```bash
docker volume prune [OPTIONS]
```

**Common Options:**
- `-f, --force`: Do not prompt for confirmation
- `--filter`: Provide filter values

**Examples:**
```bash
docker volume prune
docker volume prune -f
```

---

## System Commands

### docker version
Shows Docker version information.

```bash
docker version [OPTIONS]
```

**Common Options:**
- `-f, --format`: Pretty-print using Go template

**Examples:**
```bash
docker version
docker version --format='{{.Server.Version}}'
```

---

### docker info
Displays system-wide information.

```bash
docker info [OPTIONS]
```

**Common Options:**
- `-f, --format`: Pretty-print using Go template

**Examples:**
```bash
docker info
docker info --format='{{.Containers}}'
```

---

### docker login
Logs in to a Docker registry.

```bash
docker login [OPTIONS] [SERVER]
```

**Common Options:**
- `-u, --username`: Username
- `-p, --password`: Password
- `--password-stdin`: Read password from stdin

**Examples:**
```bash
docker login
docker login -u dockerhubusername
docker login localhost:5000
echo $PASSWORD | docker login -u dockerhubusername --password-stdin
```

---

### docker logout
Logs out from a registry.

```bash
docker logout [SERVER]
```

**Examples:**
```bash
docker logout
docker logout localhost:5000
```

---

### docker system df
Shows Docker disk usage.

```bash
docker system df [OPTIONS]
```

**Common Options:**
- `-v, --verbose`: Show detailed information

**Examples:**
```bash
docker system df
docker system df -v
```

---

### docker system prune
Removes unused Docker objects.

```bash
docker system prune [OPTIONS]
```

**Common Options:**
- `-a, --all`: Remove all unused images
- `-f, --force`: Do not prompt for confirmation
- `--volumes`: Prune volumes

**Examples:**
```bash
docker system prune
docker system prune -f
docker system prune -a
docker system prune -a --volumes
```

---

### docker events
Streams real-time Docker events.

```bash
docker events [OPTIONS]
```

**Common Options:**
- `-f, --filter`: Filter events
- `--format`: Pretty-print using Go template
- `--since`: Show events since timestamp
- `--until`: Show events until timestamp

**Examples:**
```bash
docker events
docker events --filter "type=container"
docker events --filter "event=start"
docker events --since 2025-02-01T00:00:00
```

---

## Docker Compose Commands

### docker-compose up
Builds and starts all services.

```bash
docker-compose up [OPTIONS] [SERVICE...]
```

**Common Options:**
- `-d, --detach`: Detached mode
- `--build`: Build images before starting containers
- `--no-build`: Do not build images
- `--remove-orphans`: Remove containers not defined in compose file
- `-V, --renew-anon-volumes`: Recreate anonymous volumes

**Examples:**
```bash
docker-compose up
docker-compose up -d
docker-compose up --build
docker-compose up -d web db
```

---

### docker-compose down
Stops and removes all containers.

```bash
docker-compose down [OPTIONS]
```

**Common Options:**
- `-v, --volumes`: Remove volumes
- `--rmi`: Remove images (all, local, none)
- `--remove-orphans`: Remove containers not defined in compose file

**Examples:**
```bash
docker-compose down
docker-compose down -v
docker-compose down --rmi all
```

---

### docker-compose build
Builds images for services.

```bash
docker-compose build [OPTIONS] [SERVICE...]
```

**Common Options:**
- `--no-cache`: Do not use cache
- `--force-rm`: Remove intermediate containers
- `--pull`: Always pull base images

**Examples:**
```bash
docker-compose build
docker-compose build web
docker-compose build --no-cache
```

---

### docker-compose ps
Lists services and status.

```bash
docker-compose ps [OPTIONS] [SERVICE...]
```

**Common Options:**
- `-q, --quiet`: Only display IDs
- `-a, --all`: Show all services

**Examples:**
```bash
docker-compose ps
docker-compose ps -a
docker-compose ps web
```

---

### docker-compose logs
Displays service logs.

```bash
docker-compose logs [OPTIONS] [SERVICE...]
```

**Common Options:**
- `-f, --follow`: Follow log output
- `--tail`: Last n lines of logs
- `-t, --timestamps`: Show timestamps
- `--no-log-prefix`: Do not print service names

**Examples:**
```bash
docker-compose logs
docker-compose logs -f
docker-compose logs --tail 50 web
docker-compose logs -f web db
```

---

### docker-compose exec
Executes command in service.

```bash
docker-compose exec [OPTIONS] SERVICE COMMAND [ARGS...]
```

**Common Options:**
- `-d, --detach`: Detached mode
- `-i, --interactive`: Keep STDIN open
- `-t, --tty`: Allocate pseudo-TTY
- `-u, --user`: Username or UID

**Examples:**
```bash
docker-compose exec web /bin/bash
docker-compose exec -it web /bin/sh
docker-compose exec web npm start
docker-compose exec db psql -U postgres
```

---

### docker-compose start
Starts services.

```bash
docker-compose start [OPTIONS] [SERVICE...]
```

**Examples:**
```bash
docker-compose start
docker-compose start web db
```

---

### docker-compose stop
Stops services.

```bash
docker-compose stop [OPTIONS] [SERVICE...]
```

**Common Options:**
- `-t, --timeout`: Shutdown timeout

**Examples:**
```bash
docker-compose stop
docker-compose stop web db
docker-compose stop -t 30
```

---

### docker-compose restart
Restarts services.

```bash
docker-compose restart [OPTIONS] [SERVICE...]
```

**Examples:**
```bash
docker-compose restart
docker-compose restart web
```

---

### docker-compose rm
Removes services.

```bash
docker-compose rm [OPTIONS] [SERVICE...]
```

**Common Options:**
- `-f, --force`: Do not prompt
- `-v, --volumes`: Remove volumes
- `-s, --stop`: Stop services before removing

**Examples:**
```bash
docker-compose rm
docker-compose rm -f web
docker-compose rm -f -v
```

---

### docker-compose pull
Pulls service images.

```bash
docker-compose pull [OPTIONS] [SERVICE...]
```

**Examples:**
```bash
docker-compose pull
docker-compose pull web db
```

---

### docker-compose push
Pushes service images.

```bash
docker-compose push [OPTIONS] [SERVICE...]
```

**Examples:**
```bash
docker-compose push
docker-compose push web
```

---

### docker-compose scale
Scales services to multiple instances.

```bash
docker-compose scale [SERVICE=NUM...]
```

**Examples:**
```bash
docker-compose scale web=3
docker-compose scale web=3 db=2
```

---

### docker-compose config
Validates and shows compose file.

```bash
docker-compose config [OPTIONS]
```

**Common Options:**
- `--resolve-image-digests`: Pin image versions to digests
- `-q, --quiet`: Only validate

**Examples:**
```bash
docker-compose config
docker-compose config -q
```

---

## Helpful Aliases and Functions

```bash
# Remove all stopped containers
docker rm $(docker ps -a -q)

# Remove all dangling images
docker rmi $(docker images -f "dangling=true" -q)

# Remove all unused images
docker rmi $(docker images -q)

# Stop all running containers
docker stop $(docker ps -q)

# Remove stopped containers and dangling images
docker system prune

# Get container IP
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' CONTAINER

# Monitor container in real-time
docker stats CONTAINER

# View all available networks
docker network ls

# Get total disk usage
docker system df

# Kill all running containers
docker kill $(docker ps -q)

# Access container shell
docker exec -it CONTAINER /bin/bash
docker exec -it CONTAINER /bin/sh

# View container environment variables
docker exec CONTAINER env
```

---

## Notes

- Commands are case-sensitive
- Use `docker COMMAND --help` for detailed help on any command
- For production, always use specific image tags (avoid `latest`)
- Monitor resource usage with `docker stats` to optimize performance
- Use named volumes for persistent data instead of binding host directories
- Create custom networks for better container communication
- Implement proper logging strategies for debugging
- Always clean up unused resources to save disk space

# containerd – Docker’s Container Runtime

## Overview

containerd is an industry-standard **container runtime** used by Docker and Kubernetes to manage the complete container lifecycle on a host.[web:36][web:44] It runs as a long-lived daemon and provides APIs for higher-level tools (like dockerd, CRI, or custom clients) to create, start, stop, and delete containers, and to manage images and snapshots.[web:36][web:47]

At a high level:

- Docker CLI → talks to **dockerd** (Docker daemon).
- **dockerd** → talks to **containerd** to manage containers and images.
- **containerd** → talks to a low-level OCI runtime such as **runc** to actually start container processes.[web:34][web:37][web:47]

## Role in Docker Architecture

- **Docker CLI (`docker`)**: Sends user commands (`docker run`, `docker ps`, etc.) to the Docker Engine API.[web:38]
- **Docker daemon (`dockerd`)**: Exposes the Docker API, orchestrates builds, networks, volumes, logging, and registry auth, and delegates low-level lifecycle work to containerd.[web:34][web:38][web:41]
- **containerd daemon (`containerd`)**: Runs as a separate daemon, exposes a gRPC API, and manages images, snapshots, and container tasks.[web:44][web:47]
- **OCI runtime (`runc`, etc.)**: Invoked by containerd to perform the actual kernel calls to create containers using namespaces and cgroups.[web:47][web:49]

## Key Concepts in containerd

- **Namespaces**: API-level isolation of images, containers, and snapshots inside containerd (e.g., `k8s.io`, `default`).[web:47]
- **Images**: Stored following the OCI image spec; containerd pulls and stores images in its image store.[web:35][web:47]
- **Snapshots**: Copy‑on‑write filesystem layers (overlayfs etc.) used for efficient layered filesystems and container root filesystems.[web:47][web:50]

## Typical Lifecycle via containerd

From a client’s perspective:[web:45][web:47]

1. Connect to containerd (`/run/containerd/containerd.sock` on Linux).[web:44][web:49]
2. Pull an image into the image store.
3. Create a **container** (OCI spec: rootfs, process, env, mounts, namespaces, cgroups).[web:47]
4. Create a **task** (the runtime process) from the container.
5. Start the task (containerd invokes runc to spawn the process).[web:47]
6. Monitor, signal, and eventually delete the task and cleanup resources.[web:45][web:47]

## containerd and Kubernetes

Kubernetes now commonly uses containerd directly via the Container Runtime Interface (CRI):[web:48][web:49]

- containerd exposes a CRI plugin (`io.containerd.grpc.v1.cri`).[web:49][web:52]
- Typical CRI endpoints:
  - Linux: `/run/containerd/containerd.sock`.[web:49]
  - Windows: `npipe://./pipe/containerd-containerd`.[web:49]
- With systemd, configure runc with `SystemdCgroup = true` in `config.toml` for proper cgroup v2 behavior.[web:49]

## Configuration Highlights

Main config file (Linux): `/etc/containerd/config.toml`.[web:49][web:52]

Key fields:

- `root` and `state`: Data and runtime state directories.[web:50][web:52]
- `address`: gRPC socket, e.g. `"/run/containerd/containerd.sock"`.[web:49][web:50]
- Runtimes (example for runc with systemd cgroups):[web:49][web:50][web:51]

  ```toml
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
    runtime_type = "io.containerd.runc.v2"

    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
      SystemdCgroup = true
