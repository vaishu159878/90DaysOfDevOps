# Day 32 – Docker Volumes & Networking

---

# Task 1 – Data Loss Without Volumes



## Commands Used

docker run -d --name postgres-db -e POSTGRES_PASSWORD=pass123 -p 5432:5432 postgres
docker exec -it postgres-db psql -U postgres


Created a sample database and table.


CREATE DATABASE devops;

\c devops

CREATE TABLE students(id SERIAL PRIMARY KEY,name TEXT);

INSERT INTO students(name)VALUES ('Vaishnavi');

SELECT * FROM students;


Removed the container.


docker stop postgres-db && docker rm postgres-db


Created a new PostgreSQL container.

Result:

The database and table were missing.

### Observation

Docker stores data inside the writable layer of the container.

When the container is deleted, its filesystem is deleted too.

That is why all database data was lost.

---

# Task 2 – Named Volumes

## Create Volume


docker volume create postgres-data

Verify

docker volume ls

Run PostgreSQL with volume.

docker run -d --name postgres-db -e POSTGRES_PASSWORD=pass123 -v postgres-data:/var/lib/postgresql -p 5432:5432 postgres


Again created the database and inserted data.

Removed the container.

docker stop postgres-db && docker rm postgres-db


Started a brand new container using the same volume.

docker run -d --name postgres-new -e POSTGRES_PASSWORD=pass123 -v postgres-data:/var/lib/postgresql -p 5432:5432 postgres

Connected to PostgreSQL.

docker exec -it postgres-new psql -U postgres

Verified the data.

Result:

The database and table were still available.

### Volume Inspection

docker volume inspect postgres-data

### Observation

Named volumes store data outside the container.

Removing a container does not remove the volume.

Data remains available until the volume itself is deleted.

---

# Task 3 – Bind Mounts

Created a folder on the host.

```bash
mkdir docker

cd docker
```

Created an HTML file.

```html
<h1>Hello Docker...</h1>
```

Started Nginx.

docker run -d --name nginx-bm -p 8080:80 -v $(pwd):/usr/share/nginx/html nginx

Opened

http://13.63.157.248:8080/


Edited the HTML file on the host.

Refreshed the browser.

Changes appeared instantly.

### Observation

Bind mounts directly connect a host directory with a container directory.

Changes made on the host are immediately reflected inside the container.


# Named Volume vs Bind Mount

| Named Volume              | Bind Mount                               |
|---------------------------|------------------------------------------|
| Managed by Docker         | Managed by Host OS                       |
| Best for database storage | Best for source code                     |
| Portable                  | Depends on host path                     |
| More secure               | Gives container access to host directory |


# Task 4 – Docker Networking Basics

List Docker networks.

docker network ls


Inspect bridge network.


docker network inspect bridge

Started two Ubuntu containers.


docker run -dit --name ubuntu1 ubuntu

docker run -dit --name ubuntu2 ubuntu


Installed ping utility.

apt update

apt install iputils-ping -y


Ping by name.

ping ubuntu2


Result

Failed.

Ping by IP.


ping 172.17.0.5


Result

Successful.

### Observation

Containers connected to the default bridge network cannot resolve each other by container name.

Communication works only through IP addresses.

---

# Task 5 – Custom Bridge Network

Create network.

docker network create my-app-net


Run containers.

docker run -dit --name myapp --network my-app-net ubuntu

docker run -dit --name myapp2 --network my-app-net ubuntu

Ping by name.

ping myapp2


Result

Successful.

### Observation

Custom bridge networks provide automatic DNS resolution.

Containers can communicate using container names without knowing IP addresses.

---

# Task 6 – Database + Application

Created custom network.

```bash
docker network create app-network
```

Started PostgreSQL.

docker run -d --name new-postgres --network app-network -v postgres-data:/var/lib/postgresql -e POSTGRES_PASSWORD=pass123 postgre

Started Ubuntu container.

docker run -dit --name app-container --network app-network ubuntu

Installed ping.


apt update

apt install iputils-ping -y


Verified connectivity.


ping new-postgres


Result

Successful.

Application container successfully reached the database using the container name.

---

# What I Learned

✅ Containers are temporary by default.

✅ Data inside containers is lost when the container is removed.

✅ Docker Volumes provide persistent storage.

✅ Bind Mounts connect host files directly to containers.

✅ Default bridge network supports IP communication but not automatic name resolution.

✅ Custom bridge networks provide built-in DNS for container-to-container communication.

✅ Volumes and custom networks are essential for running real-world multi-container applications.

---



