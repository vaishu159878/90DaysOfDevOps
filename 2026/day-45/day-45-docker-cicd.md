# GitHub Actions Practice

![Docker Build and Push](https://github.com/vaishu159878/github-actions-practice/actions/workflows/docker-publish.yml/badge.svg)

A hands-on repository for practicing GitHub Actions, Docker, CI/CD, and DevOps workflows.

## 🚀 Day 45 – Docker Build & Push in GitHub Actions

This project demonstrates a complete Docker CI/CD pipeline:

```text
Git Push
   ↓
GitHub Actions
   ↓
Docker Build
   ↓
Docker Hub
   ↓
Ubuntu EC2
   ↓
Docker Container
   ↓
Nginx Application
```

## 🐳 Docker Image

Docker Hub repository:

**vaishnavinalawade/github-actions-practice**

Available tags:

- `latest`
- `sha-<short-commit-hash>`

## ⚙️ GitHub Actions Workflow

Workflow file:

```text
.github/workflows/docker-publish.yml
```

The workflow:

1. Checks out the repository.
2. Sets up Docker Buildx.
3. Logs in to Docker Hub using GitHub Secrets.
4. Builds the Docker image.
5. Creates a `latest` tag.
6. Creates a commit-SHA tag.
7. Pushes both tags to Docker Hub when code is pushed to `main`.

## 🌿 Branch Protection Behavior

Feature branches can build the Docker image, but Docker Hub publishing is restricted to `main`.

```text
Feature branch → Docker Build ✅ → Docker Push ❌

Main branch    → Docker Build ✅ → Docker Push ✅
```

## 🔐 GitHub Secrets

The workflow uses these repository secrets:

```text
DOCKER_USERNAME
DOCKER_TOKEN
```

The Docker Hub access token is stored securely in GitHub and is never hard-coded in the workflow.

## 🐋 Dockerfile

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

## 🖥️ Run the Image on Ubuntu EC2

Pull the image:

```bash
docker pull vaishnavinalawade/github-actions-practice:latest
```

Run the container:

```bash
docker run -d   --name day45-app   -p 8080:80   vaishnavinalawade/github-actions-practice:latest
```

Check the container:

```bash
docker ps
```

Test the application:

```bash
curl http://localhost:8080
```

Open it from a browser using:

```text
http://<EC2-PUBLIC-IP>:8080
```

Make sure port `8080` is allowed in the EC2 security group's inbound rules.

## 📁 Project Structure

```text
github-actions-practice/
├── .github/
│   └── workflows/
│       └── docker-publish.yml
├── 2026/
│   └── day-45/
│       └── day-45-docker-cicd.md
├── Dockerfile
├── index.html
└── README.md
```

## 📚 What I Practiced

- Docker image building
- Dockerfile
- Docker containers
- GitHub Actions
- Docker Buildx
- Docker Hub authentication
- GitHub Secrets
- Docker image tagging
- Commit SHA based versioning
- CI/CD workflow
- Feature branch testing
- Docker image publishing
- Pulling images on Ubuntu EC2
- Running containers on EC2

## 🔄 Complete CI/CD Journey

```text
Developer
    ↓
git push
    ↓
GitHub Repository
    ↓
GitHub Actions
    ↓
Checkout Code
    ↓
Docker Buildx
    ↓
Build Docker Image
    ↓
Docker Hub Login
    ↓
Push latest + SHA Tags
    ↓
Docker Hub
    ↓
docker pull
    ↓
Ubuntu EC2
    ↓
docker run
    ↓
Running Application
```


