# Day 35 – Multi-Stage Builds & Docker Hub


# Task 1 – Single Stage Build


### app.js

console.log("Hello from Docker Multi-Stage Build!");


### package.json

{
  "name": "docker-multistage-demo",
  "version": "1.0.0",
  "main": "app.js"
}

---

## Dockerfile (Single Stage)

FROM node:22

WORKDIR /app

COPY . .

CMD ["node", "app.js"]

---

## Build Image

docker build -f Dockerfile.single -t node-single:v1 .

Check image size

docker images

Example

| Image          | Size   |
|----------------|--------|
| node-single:v1 | 1.62GB |

---

# Task 2 – Multi-Stage Build

## Dockerfile (Multi Stage)

FROM node:22 AS builder

WORKDIR /app

COPY . .

FROM node:22-alpine

WORKDIR /app

COPY --from=builder /app .

CMD ["node", "app.js"]


Build image:

docker build -t node-multistage:v1 .


Check size:

docker images

Example

| Image              | Size  |
|--------------------|-------|
| node-multistage:v1 | 230MB |

---

## Comparison

| Build Type   | Image Size |
|--------------|------------|
| Single Stage | 1.62GB     |
| Multi Stage  | 230MB      |

### Why is the Multi-Stage Image Smaller?

- Build dependencies are not included.
- Only the required application files are copied.
- Uses a lightweight Alpine image.
- Reduces attack surface.
- Faster to pull and deploy.

---

# Task 3 – Push Image to Docker Hub

Login:

docker login

Tag image:


docker tag node-multistage:v1 vaishnavinalawade/node-multistage:v1

Push image:

docker push vaishnavinalawade/node-multistage:v1


Verify:

docker image rm vaishnavinalawade/node-multistage:v1

docker pull vaishnavinalawade/node-multistage:v1


Run:

docker run vaishnavinalawade/node-multistage:v1


Output

Hello from Docker Multi-Stage Build!


---

# Task 4 – Docker Hub Repository

Pull latest

docker pull vaishnavinalawade/node-multistage:latest


Pull a specific version

docker pull vaishnavinalawade/node-multistage:v1


# Task 5 – Image Best Practices

## Use Alpine

FROM node:22-alpine

Smaller than Ubuntu.

---

RUN adduser -D appuser

USER appuser

Improves security.

---

## Use Specific Tags

Good

FROM node:22-alpine

Avoid

FROM node:latest

Specific tags improve reproducibility.

---
