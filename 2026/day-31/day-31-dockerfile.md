# Day 31 – Dockerfile: Build Your Own Images

---

## Task 1: My First Dockerfile

I created a Dockerfile using Ubuntu as the base image and installed curl inside the image.

### Dockerfile

```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get install -y curl

CMD ["echo", "Hello from my custom image!"]
```

### Build Image

```bash
docker build -t my-ubuntu:v1 .
```

### Run Container

```bash
docker run my-ubuntu:v1
```

### Output

```text
Hello from my custom image!
```

### What I Learned

* Docker images are built from Dockerfiles.
* The `RUN` instruction executes commands while building the image.
* The `CMD` instruction runs when the container starts.

---

## Task 2: Dockerfile Instructions

I created another Dockerfile to understand common Dockerfile instructions.

### Dockerfile

```dockerfile
FROM nginx:alpine

WORKDIR /usr/share/nginx/html

COPY index.html .

RUN echo "Dockerfile demo image created"

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Build

```bash
docker build -t docker-demo:v1 .
```

### What I Learned

* `FROM` selects the base image.
* `WORKDIR` sets the working directory.
* `COPY` copies files into the image.
* `RUN` executes commands during build.
* `EXPOSE` documents the port used by the application.
* `CMD` defines the default command.

---

## Task 3: CMD vs ENTRYPOINT

### CMD Example

```dockerfile
FROM alpine

CMD ["echo", "Hello..!"]
```

Run:

```bash
docker run cmd-demo
```

Output:

```text
Hello..!
```

### ENTRYPOINT Example

```dockerfile
FROM alpine

ENTRYPOINT ["echo"]
```

Run:

```bash
docker run entrypoint-demo hello
```

Output:

```text
hello
```

### My Understanding

* CMD provides a default command and can be overridden.
* ENTRYPOINT always runs and accepts additional arguments.
* CMD is useful for defaults, while ENTRYPOINT is useful when the container should always perform one specific action.

---

## Task 4: Static Website with Nginx

I created a simple HTML page and served it using Nginx inside a Docker container.

### index.html

```html
<h1>Hello from Docker!</h1>
<p>Day 31 of #90DaysOfDevOps</p>
```

### Dockerfile

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html
```

### Build

```bash
docker build -t my-website:v1 .
```

### Run

```bash
docker run -d -p 8080:80 my-website:v1
```

### Result

I was able to access the webpage from my browser using the EC2 public IP and port 8080.

---

## Task 5: Using .dockerignore

I created a `.dockerignore` file to exclude unnecessary files from the build context.

### .dockerignore

```text
node_modules
.git
*.md
.env
```

### What I Learned

Using `.dockerignore` helps:

* Reduce build context size
* Improve build speed
* Avoid copying unnecessary files
* Prevent sensitive files from being included in images

---

## Task 6: Docker Build Cache

I rebuilt the image multiple times and noticed Docker reused existing layers.

### Observation

```text
Using cache
```

appeared during the build process.

### What I Learned

Docker stores layers from previous builds. If a layer has not changed, Docker reuses it instead of rebuilding it.

This makes image builds much faster.

---

## Commands Practiced

```bash
docker build
docker run
docker images
docker ps
docker ps -a
```

---

