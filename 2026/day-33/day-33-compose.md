# Day 33 – Docker Compose: Multi-Container Basics

## Objective

Learn how to use Docker Compose to manage multi-container applications using a single YAML configuration file.

---

# Task 1: Install & Verify Docker Compose

### Check Docker Compose


docker compose version

### Output

Docker Compose version 2.40.3+ds1-0ubuntu1

---

# Task 2: First Docker Compose Project


## docker-compose.yml


services:
  nginx:
    image: nginx
    container_name: compose-nginx
    ports:
      - "8080:80"


## Start Container


docker compose up -d


## Verify

Open browser:


http://13.60.218.6:8080


Nginx welcome page should appear.

## Stop Project


docker compose down

---

# Task 3: WordPress + MySQL using Docker Compose


## .env

```env
MYSQL_ROOT_PASSWORD=rootpassword
MYSQL_DATABASE=wordpress
MYSQL_USER=wpuser
MYSQL_PASSWORD=wppassword
```

## docker-compose.yml

services:
  mysql:
    image: mysql:8.0
    container_name: mysql-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - mysql_data:/var/lib/mysql

  wordpress:
    image: wordpress
    container_name: wordpress-app
    restart: always
    ports:
      - "8081:80"
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_USER: ${MYSQL_USER}
      WORDPRESS_DB_PASSWORD: ${MYSQL_PASSWORD}
      WORDPRESS_DB_NAME: ${MYSQL_DATABASE}
    depends_on:
      - mysql

volumes:
  mysql_data:


## Start Application

docker compose up -d


## Verify

Visit

```
http://13.60.218.6:8081
```

Complete WordPress installation.

---

## Data Persistence Test

Stop services

docker compose down


Start again

docker compose up -d


WordPress database data remains available because MySQL uses a named Docker volume.

---

# Task 4: Docker Compose Commands

## Start in Detached Mode

docker compose up -d


## View Running Services

docker compose ps


## View All Logs


docker compose logs


Follow logs


docker compose logs -f


## Logs for Specific Service

docker compose logs wordpress

or

docker compose logs mysql

## Stop Containers

docker compose stop


## Remove Containers and Network

docker compose down

## Rebuild Images

docker compose up --build

---

# Task 5: Environment Variables

Variables stored inside `.env`

```env
MYSQL_DATABASE=wordpress
MYSQL_USER=wpuser
MYSQL_PASSWORD=wppassword
MYSQL_ROOT_PASSWORD=rootpassword
```

Compose automatically reads variables using:

environment:
  MYSQL_DATABASE: ${MYSQL_DATABASE}


## Verify Variables

docker compose config


This command displays the final Compose configuration after substituting environment variables.

---



