# Day 37 – Docker Revision

## Goal

Revise Docker concepts and commands from Days 29–36 and identify areas that need more hands-on practice.

---

# Self-Assessment Checklist

| Topic                                   | Status   |
| --------------------------------------- | -------- |
| Run a container from Docker Hub         | ✅ Can do |
| List, stop and remove containers/images | ✅ Can do |
| Explain image layers and caching        | ✅ Can do |
| Write a Dockerfile from scratch         | ✅ Can do |
| Explain CMD vs ENTRYPOINT               | ✅ Can do |
| Build and tag a custom image            | ✅ Can do |
| Create and use named volumes            | ✅ Can do |
| Use bind mounts                         | ✅ Can do |
| Create custom networks                  | ✅ Can do |
| Connect containers using networks       | ✅ Can do |
| Write a docker-compose.yml              | ⚠️ Shaky |
| Use environment variables and `.env`    | ⚠️ Shaky |
| Write a multi-stage Dockerfile          | ⚠️ Shaky |
| Push an image to Docker Hub             | ✅ Can do |
| Use healthchecks and depends_on         | ⚠️ Shaky |

## Weak Areas Selected

1. Docker Compose
2. Multi-stage Docker builds

I will revisit these topics with hands-on practice instead of only reading the commands.

---

# Quick-Fire Questions

## 1. What is the difference between an image and a container?

An **image** is a read-only template containing the application, dependencies and filesystem layers.

A **container** is a running or stopped instance created from an image.

```text
Docker Image
     ↓
Container
```

---

## 2. What happens to data inside a container when you remove it?

Data stored only inside the container's writable layer is removed when the container is deleted.

For persistent data, use a **named volume** or **bind mount**.

Example:

```bash
docker run -v mydata:/data nginx
```

The volume can survive container deletion.

---

## 3. How do two containers on the same custom network communicate?

They communicate using the **container/service name** through Docker's internal DNS.

Example:

```text
app → db:5432
```

Instead of depending on an IP address, the application can connect to:

```text
db
```

---

## 4. What does `docker compose down -v` do differently from `docker compose down`?

```bash
docker compose down
```

Stops and removes Compose-created containers and networks.

```bash
docker compose down -v
```

Also removes the volumes associated with the Compose application.

Therefore, `down -v` can delete persistent application data stored in those volumes.

---

## 5. Why are multi-stage builds useful?

Multi-stage builds separate the **build environment** from the **runtime environment**.

Benefits:

* Smaller production images
* Fewer unnecessary dependencies
* Better security
* Cleaner production containers
* Faster image transfer/deployment

Example:

```dockerfile
FROM node:22 AS builder

# Build application here

FROM nginx:alpine

# Copy only build output
COPY --from=builder /app/dist /usr/share/nginx/html
```

---

## 6. What is the difference between `COPY` and `ADD`?

`COPY` simply copies files/directories into the image.

`ADD` can also handle features such as extracting local archives.

For normal file copying, **prefer `COPY`** because its behavior is simpler and more explicit.

---

## 7. What does `-p 8080:80` mean?

```bash
-p 8080:80
```

means:

```text
Host port     Container port
   8080   →       80
```

So a request to:

```text
localhost:8080
```

can reach port `80` inside the container.

---

## 8. How do you check how much disk space Docker is using?

Use:

```bash
docker system df
```

For more detailed information:

```bash
docker system df -v
```

---

# Important Docker Concepts

## Image Layers

Docker images are built from layers.

For example:

```dockerfile
FROM ubuntu
RUN apt-get update
COPY app.py /app/
```

Each filesystem-changing instruction can create a layer.

Docker can reuse unchanged layers during subsequent builds, which makes builds faster.

---

## Build Cache

If an instruction and everything it depends on hasn't changed, Docker can reuse its cached result.

A common optimization is:

```dockerfile
COPY package*.json ./
RUN npm install

COPY . .
```

Instead of:

```dockerfile
COPY . .
RUN npm install
```

This allows dependency installation to remain cached when only application source code changes.

---

# Hands-On Revision

## 1. Test Docker Container

```bash
docker run -d --name revision-nginx -p 8080:80 nginx
docker ps
docker logs revision-nginx
curl http://localhost:8080
docker stop revision-nginx
docker rm revision-nginx
```

---

## 2. Test Named Volume

```bash
docker volume create revision-data

docker run -it --name volume-test \
  -v revision-data:/data \
  ubuntu bash
```

Inside the container:

```bash
echo "Docker Revision" > /data/test.txt
cat /data/test.txt
exit
```

Remove the container:

```bash
docker rm volume-test
```

Create another container using the same volume:

```bash
docker run --rm \
  -v revision-data:/data \
  ubuntu cat /data/test.txt
```

The data should still exist.

---

## 3. Test Custom Network

```bash
docker network create revision-net
```

Run two containers:

```bash
docker run -d --name container1 --network revision-net nginx
docker run -it --rm --network revision-net busybox sh
```

Inside BusyBox:

```bash
ping container1
```

The container name can be resolved through Docker's internal DNS.

Clean up:

```bash
docker rm -f container1
docker network rm revision-net
```

---

# Final Revision Checklist

Before moving forward, I should be able to explain:

* [x] Image vs container
* [x] Container lifecycle
* [x] Image layers
* [x] Docker build cache
* [x] Dockerfile instructions
* [x] CMD vs ENTRYPOINT
* [x] Port mapping
* [x] Named volumes
* [x] Bind mounts
* [x] Docker networks
* [x] Container DNS
* [x] Docker Compose
* [x] Environment variables
* [x] Multi-stage builds
* [x] Healthchecks
* [x] Docker cleanup
* [x] Docker Hub push workflow

---

# Day 37 Takeaway

Docker is not just about running containers.

The important concepts are:

```text
Image
  ↓
Container
  ↓
Network
  ↓
Storage
  ↓
Compose
  ↓
Production-ready application
```

The goal of this revision is not to memorize every Docker command.

The goal is to understand **why and when to use each Docker feature**.

---

## Next Step

I will continue practicing the weak areas, especially:

1. Docker Compose
2. Multi-stage builds

Then I can move forward with confidence to the next DevOps topic.
