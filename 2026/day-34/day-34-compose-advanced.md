# Day 34 - Docker Compose Advanced


## Task 1 - Multi-Container Application

I created a Docker Compose file with three services.

- Flask application
- PostgreSQL database
- Redis cache

Instead of using a pre-built image for the Flask app, I created my own Dockerfile and built the image using Docker Compose.

**Status:** ✅ Completed

---

## Task 2 - depends_on & Healthcheck

I added a health check for the PostgreSQL container using `pg_isready`.

Then I used:

depends_on:
  db:
    condition: service_healthy

This made sure my Flask application started only after the database became healthy.

I verified it using:

docker compose ps


The PostgreSQL container showed **healthy** before the web application started.

---

## Task 3 - Restart Policy

I configured PostgreSQL with:

restart: always


Then I tested it by killing the database container.

docker kill postgres-db


Docker automatically restarted the container.

I also learned the difference between:

- **restart: always** → Best for databases and production services.
- **restart: on-failure** → Restarts only if the container exits with an error.

---

## Task 4 - Build from Dockerfile

Instead of pulling an application image from Docker Hub, I built my own image.

Whenever I changed the code, I rebuilt everything using:


docker compose up --build


This made development much easier.


---

## Task 5 - Networks & Volumes

I created:

- A custom bridge network
- A named volume for PostgreSQL data

I also added labels to organize the services.

Using a named volume means the database data remains safe even if the container is removed.


---

## Task 6 - Scaling (Bonus)

I tried scaling my Flask application.

docker compose up --scale web=3


At first, Docker didn't allow scaling because I had set a fixed `container_name`.

After understanding that issue, I also learned that multiple containers cannot use the same host port.

Now I understand why production environments use load balancers or reverse proxies instead of exposing every container directly.

Even though scaling didn't fully work, I understood why it failed, and that was the biggest learning from today's task.



---

