# Day 57 – Resource Requests, Limits, and Probes

## Objective

## 1. Resource Requests and Limits

I created a Pod with the following resources:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "250m"
    memory: "256Mi"
```

### Requests

Requests tell the Kubernetes scheduler how much CPU and memory the Pod needs for scheduling.

```text
CPU:    100m
Memory: 128Mi
```

### Limits

Limits define the maximum resources the container can use.

```text
CPU:    250m
Memory: 256Mi
```

### QoS Class

Since the requests and limits are configured but are different, Kubernetes assigned:

```text
QoS Class: Burstable
```

---

## 2. OOMKilled – Exceeding Memory Limit

For this test, I used the `polinux/stress` image with a memory limit of `100Mi`.

The container was configured to allocate approximately `200M` of memory.

```yaml
resources:
  limits:
    memory: "100Mi"
```

The container exceeded its memory limit and Kubernetes killed it.

I observed:

```text
Reason: OOMKilled
Exit Code: 137
```

The Pod also restarted because the Pod's restart policy caused Kubernetes to start the container again.

### Why 137?

Exit code `137` represents:

```text
128 + 9 = 137
```

Signal `9` is `SIGKILL`.

---

## 3. Pending Pod – Insufficient Resources

I created a Pod requesting:

```yaml
resources:
  requests:
    cpu: "100"
    memory: "128Gi"
```

The Kubernetes cluster could not satisfy these resource requests.

The Pod remained:

```text
STATUS: Pending
```

The scheduler reported:

```text
FailedScheduling

Insufficient cpu
Insufficient memory
```

This helped me understand that **resource requests affect scheduling**.

Kubernetes does not place a Pod on a node unless the node has enough allocatable resources to satisfy its requests.

---

## 4. Liveness Probe

A liveness probe checks whether a running container is still healthy.

I created a BusyBox container that:

1. Created `/tmp/healthy`
2. Waited 30 seconds
3. Deleted `/tmp/healthy`

The liveness probe checked the file using:

```yaml
livenessProbe:
  exec:
    command:
      - cat
      - /tmp/healthy
  periodSeconds: 5
  failureThreshold: 3
```

After the file was deleted, the probe failed three consecutive times.

Kubernetes detected that the container was unhealthy and restarted it.

I observed:

```text
RESTARTS: 1
```

### Key point

**Liveness failure → container restart**

---

## 5. Readiness Probe

A readiness probe checks whether a Pod is ready to receive traffic.

I used an Nginx container with an HTTP readiness probe:

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
  periodSeconds: 5
  failureThreshold: 3
```

I exposed the Pod using a Service:

```bash
kubectl expose pod readiness-pod --port=80 --name=readiness-svc
```

Initially, the Pod was ready and its IP appeared in the Service endpoints.

Then I removed the Nginx `index.html` file:

```bash
kubectl exec readiness-pod -- rm /usr/share/nginx/html/index.html
```

After the readiness probe failed:

```text
READY: 0/1
RESTARTS: 0
```

The Service endpoints became empty.

This demonstrated that a readiness failure **does not restart the container**.

Instead, Kubernetes removes the Pod from the Service's available endpoints.

### Key point

**Readiness failure → remove Pod from Service endpoints**

## 6. Startup Probe

A startup probe is useful for applications that need extra time to start.

I created a container that waited 20 seconds before creating:

```text
/tmp/started
```

The startup probe checked for this file:

```yaml
startupProbe:
  exec:
    command:
      - cat
      - /tmp/started
  periodSeconds: 5
  failureThreshold: 12
```

This provided a startup budget of approximately:

```text
12 × 5 seconds = 60 seconds
```

Since the application started in about 20 seconds, the startup probe eventually succeeded.

I observed:

```text
0/1 Running
        ↓
1/1 Running
```

with:

```text
RESTARTS: 0
```

The initial startup probe failures were expected because `/tmp/started` did not exist yet.

### What if `failureThreshold` was 2?

With:

```text
periodSeconds: 5
failureThreshold: 2
```

the startup budget would be approximately:

```text
2 × 5 = 10 seconds
```

But the application needs around 20 seconds to start.

Therefore, Kubernetes would consider the startup probe failed before the application finished starting and the container would be restarted.

### Key point

**Startup probe → gives slow applications time to initialize**

---

## 7. Liveness vs Readiness vs Startup

| Probe     | Purpose                              | When it fails                           |
| --------- | ------------------------------------ | --------------------------------------- |
| Liveness  | Checks if container is still healthy | Container is restarted                  |
| Readiness | Checks if Pod can receive traffic    | Pod is removed from Service endpoints   |
| Startup   | Checks if application has started    | Container is restarted if startup fails |

A simple way I remember them:

```text
Startup  → "Has my application started?"
Liveness → "Is my application still alive?"
Readiness → "Can my application receive traffic?"
```

---

## 8. CPU vs Memory Limits

One important concept I learned today is that CPU and memory behave differently.

### CPU

CPU is **compressible**.

If a container tries to use more CPU than its limit, Kubernetes can throttle it.

```text
CPU limit exceeded
        ↓
CPU throttling
```

### Memory

Memory is **incompressible**.

If a container exceeds its memory limit, it can be killed.

```text
Memory limit exceeded
        ↓
OOMKilled
        ↓
Exit Code 137
```

---

