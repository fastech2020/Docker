# Docker Knowledge Base

## Table of Contents
1. [Docker Fundamentals](#docker-fundamentals)
2. [Core Concepts](#core-concepts)
3. [Dockerfile Instructions](#dockerfile-instructions)
4. [Docker CLI Commands](#docker-cli-commands)
5. [Docker Compose](#docker-compose)
6. [Best Practices](#best-practices)
7. [Docker Architecture](#docker-architecture)

---

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
