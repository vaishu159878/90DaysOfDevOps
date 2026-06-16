# Day 30 – Docker Images & Container Lifecycle

## What I Did Today

Today I learned how Docker images and containers work.

Until now, I was just running containers. Today I understood what happens behind the scenes.

---

## Pulled Docker Images

```bash
docker pull nginx
docker pull ubuntu
docker pull alpine
```

Checked available images:

```bash
docker images
```

### Observation

* Alpine: 13MB
* Ubuntu: 160MB
* Nginx: 241MB

I was surprised by how small Alpine is compared to Ubuntu.

Alpine contains only essential packages, which makes it lightweight and suitable for containers.

---

## Image Inspection

Used:

```bash
docker image inspect nginx
```

I could see:

* Image ID
* Creation date
* Environment variables
* Exposed ports
* Architecture

---

## Image Layers

Checked image history:

```bash
docker image history nginx
```

### What I Learned

Docker images are made of multiple layers.

Some layers showed size values while others showed 0B.

Layers with 0B only store metadata such as:

* CMD
* EXPOSE
* ENTRYPOINT

Docker uses layers to:

* Save storage
* Reuse existing layers
* Speed up image builds

---

## Container Lifecycle

Created a container:

```bash
docker create --name demo nginx
```

Started it:

```bash
docker start demo
```

Paused it:

```bash
docker pause demo
```

Unpaused it:

```bash
docker unpause demo
```

Stopped it:

```bash
docker stop demo
```

Restarted it:

```bash
docker restart demo
```

Killed it:

```bash
docker kill demo
```

Removed it:

```bash
docker rm demo
```

### States Observed

* Created
* Running
* Paused
* Exited
* Removed

---

## Working with Containers

Viewed logs:

```bash
docker logs demo
```

Followed logs:

```bash
docker logs -f demo
```

Entered container:

```bash
docker exec -it demo bash
```

Ran command without entering:

```bash
docker exec demo ls var/
```

Inspected container:

```bash
docker inspect demo
```

Found container IP address:

```text
172.17.0.2
```

---

## Cleanup

Removed unused resources:

```bash
docker system prune
```

Checked Docker disk usage:

```bash
docker system df
```

---


