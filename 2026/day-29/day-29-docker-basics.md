# Day 29 - Introduction to Docker

## What I Learned

### What is Docker?

Docker is a platform that allows applications and their dependencies to be packaged into containers. These containers can run consistently across different environments without worrying about system differences.

### Containers vs Virtual Machines

| Containers           | Virtual Machines   |
| -------------------- | ------------------ |
| Share host OS kernel | Have their own OS  |
| Lightweight          | Heavyweight        |
| Start in seconds     | Start in minutes   |
| Use fewer resources  | Use more resources |
| Portable             | Larger in size     |

### Docker Architecture

Docker consists of:

* Docker Client
* Docker Daemon (dockerd)
* Docker Images
* Docker Containers
* Docker Registry (Docker Hub)

Workflow:


Docker Client → Docker Daemon → Images/Containers
                         ↓
                    Docker Hub


---

## Practical Tasks Completed

### 1. Verified Docker Installation


docker --version


### 2. Ran My First Container

docker run hello-world


### 3. Ran an Nginx Container


docker run -d -p 80:80 --name my-nginx nginx

Accessed the Nginx welcome page using my EC2 Public IP.

### 4. Explored an Ubuntu Container


docker run -it ubuntu bash


Commands used inside the container:


pwd
ls
cat /etc/os-release


### 5. Listed Containers

Running containers:

docker ps


All containers:


docker ps -a


### 6. Viewed Container Logs


docker logs my-nginx

### 7. Executed Commands Inside Container


docker exec -it my-nginx bash

### 8. Stopped and Removed Container


docker stop my-nginx && docker rm my-nginx


---

## Screenshots

img1.png
img2.png
img3.png
img4.png



#90DaysOfDevOps #TrainWithShubham
