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

