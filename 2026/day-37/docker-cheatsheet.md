# Docker Cheat Sheet

A quick reference for commonly used Docker commands and Dockerfile instructions.

---

## 🐳 Container Commands

```bash
# Run a container
docker run nginx

# Run interactively
docker run -it ubuntu bash

# Run in detached mode
docker run -d nginx

# Run with a custom name
docker run -d --name web nginx

# List running containers
docker ps

# List all containers
docker ps -a

# Stop a container
docker stop <container>

# Start a stopped container
docker start <container>

# Restart a container
docker restart <container>

# Remove a container
docker rm <container>

# Force remove a running container
docker rm -f <container>

# Execute a command inside a running container
docker exec -it <container> bash

# View container logs
docker logs <container>

# Follow container logs
docker logs -f <container>

# Inspect a container
docker inspect <container>

# Show container resource usage
docker stats

# Map host port to container port
docker run -d -p 8080:80 nginx
```

---

## 🖼️ Image Commands

```bash
# List local images
docker images

# Pull an image from Docker Hub
docker pull nginx

# Build an image
docker build -t myapp:1.0 .

# Tag an image
docker tag myapp:1.0 username/myapp:1.0

# Remove an image
docker rmi <image>

# Force remove an image
docker rmi -f <image>

# Push an image to Docker Hub
docker push username/myapp:1.0

# Inspect an image
docker inspect <image>

# View image history/layers
docker history <image>
```

---

## 💾 Volume Commands

```bash
# Create a named volume
docker volume create mydata

# List volumes
docker volume ls

# Inspect a volume
docker volume inspect mydata

# Use a named volume
docker run -d -v mydata:/data nginx

# Remove a volume
docker volume rm mydata

# Remove unused volumes
docker volume prune
```

### Bind Mount

```bash
# Mount host directory into container
docker run -d -v $(pwd):/app nginx
```

**Named volume:** Docker manages the storage location.

**Bind mount:** You control the host path.

---

## 🌐 Network Commands

```bash
# List Docker networks
docker network ls

# Create a custom network
docker network create mynetwork

# Inspect a network
docker network inspect mynetwork

# Connect a container to a network
docker network connect mynetwork <container>

# Disconnect a container
docker network disconnect mynetwork <container>

# Run container on a specific network
docker run -d --network mynetwork --name web nginx
```

### Container-to-Container Communication

Containers on the same user-defined network can communicate using the **container/service name**.

Example:

```text
app → db:5432
```

No need to use the container's IP address.

---

## 📦 Docker Compose Commands

```bash
# Start services
docker compose up

# Start in detached mode
docker compose up -d

# Stop and remove services
docker compose down

# Stop services and remove volumes
docker compose down -v

# List Compose services
docker compose ps

# View service logs
docker compose logs

# Follow service logs
docker compose logs -f

# Build images
docker compose build

# Rebuild and start
docker compose up --build

# Restart services
docker compose restart
```

---

## 🔐 Environment Variables

### Run with an environment variable

```bash
docker run -e APP_ENV=production myapp
```

### Compose

```yaml
services:
  app:
    image: myapp:1.0
    environment:
      APP_ENV: production
```

### `.env`

```env
APP_ENV=production
DB_HOST=db
DB_PORT=5432
```

Compose can automatically load variables from a `.env` file.

---

## 🧹 Cleanup Commands

```bash
# Show Docker disk usage
docker system df

# Remove stopped containers
docker container prune

# Remove unused images
docker image prune

# Remove unused volumes
docker volume prune

# Remove unused networks
docker network prune

# Remove unused Docker resources
docker system prune

# Remove unused resources including volumes
docker system prune --volumes
```

⚠️ Use prune commands carefully because they delete unused Docker resources.

---

# Dockerfile Instructions

## FROM

Defines the base image.

```dockerfile
FROM ubuntu:24.04
```

## RUN

Executes commands while building the image.

```dockerfile
RUN apt-get update && apt-get install -y nginx
```

## COPY

Copies files from the build context into the image.

```dockerfile
COPY . /app
```

## ADD

Copies files and provides additional features such as archive extraction and URL handling.

```dockerfile
ADD app.tar.gz /app
```

Prefer `COPY` unless you specifically need an `ADD` feature.

## WORKDIR

Sets the working directory.

```dockerfile
WORKDIR /app
```

## EXPOSE

Documents the port the application listens on.

```dockerfile
EXPOSE 8080
```

It does **not** publish the port to the host.

## CMD

Provides the default command when the container starts.

```dockerfile
CMD ["python", "app.py"]
```

## ENTRYPOINT

Defines the main executable for the container.

```dockerfile
ENTRYPOINT ["python"]
```

Example:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

---

# CMD vs ENTRYPOINT

| CMD                                | ENTRYPOINT                        |
| ---------------------------------- | --------------------------------- |
| Provides default command/arguments | Defines the main executable       |
| Easy to override                   | Usually remains fixed             |
| Can provide default arguments      | CMD can provide default arguments |

Example:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Running:

```bash
docker run myapp
```

executes:

```bash
python app.py
```

---

# Multi-Stage Docker Build

Useful for creating **smaller production images**.

```dockerfile
FROM node:22 AS builder

WORKDIR /app
COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html
```

The final image contains only what is needed to run the application, rather than the complete build environment.

---

# Healthcheck

Example:

```dockerfile
HEALTHCHECK --interval=30s --timeout=5s \
  CMD curl -f http://localhost:8080/health || exit 1
```

Check container health:

```bash
docker ps
```

---

# Useful Docker Options

```bash
-d              # Detached mode
-it             # Interactive terminal
-p HOST:CONTAINER  # Port mapping
-v HOST:CONTAINER  # Volume/bind mount
--name          # Container name
-e              # Environment variable
--network       # Attach to network
--rm            # Automatically remove container after exit
```

---

## 🔑 Docker Concepts to Remember

* **Image** → Read-only template used to create containers.
* **Container** → Running/created instance of an image.
* **Volume** → Persistent Docker-managed storage.
* **Bind mount** → Maps a host path into a container.
* **Network** → Enables communication between containers.
* **Dockerfile** → Instructions for building an image.
* **Compose** → Defines and manages multi-container applications.
* **Registry** → Stores and distributes container images.
* **Multi-stage build** → Separates build and runtime environments.
* **Healthcheck** → Reports whether a containerized application is healthy.
