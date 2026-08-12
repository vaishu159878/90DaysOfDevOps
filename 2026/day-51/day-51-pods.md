# Day 51 – Kubernetes Manifests and Your First Pods

---

## 1. Kubernetes Manifest Anatomy

A Kubernetes manifest describes the desired state of a resource using YAML.

The four main top-level fields are:

### `apiVersion`

Defines the Kubernetes API version used by the resource.

For a Pod:

```yaml
apiVersion: v1
```

### `kind`

Defines the type of Kubernetes resource.

For this task:

```yaml
kind: Pod
```

### `metadata`

Contains information that identifies the resource, such as its name and labels.

Example:

```yaml
metadata:
  name: nginx-pod
  labels:
    app: nginx
```

### `spec`

Defines the desired configuration of the resource.

For a Pod, it specifies containers, images, commands, ports, and other settings.

---

## 2. Nginx Pod

I created an Nginx Pod using a YAML manifest.

### `nginx-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

Applied using:

```bash
kubectl apply -f nginx-pod.yaml
```

Verified using:

```bash
kubectl get pods
kubectl get pods -o wide
```

The Pod reached:

```text
STATUS: Running
```

### Testing Nginx from inside the Pod

I entered the container:

```bash
kubectl exec -it nginx-pod -- /bin/bash
```

Then tested the Nginx web server:

```bash
curl localhost:80
```

The response displayed the **Nginx welcome page**, confirming that Nginx was running successfully inside the Pod.

---

## 3. BusyBox Pod

I created a BusyBox Pod with a custom command to keep the container running.

### `busybox-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
  labels:
    app: busybox
    environment: dev
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["sh", "-c", "echo Hello from BusyBox && sleep 3600"]
```

Applied using:

```bash
kubectl apply -f busybox-pod.yaml
```

Checked the logs:

```bash
kubectl logs busybox-pod
```

Output:

```text
Hello from BusyBox
```

This showed how the `command` field can be used to control what a container runs and keep a container alive for testing.

---

## 4. Third Pod – Alpine

I created a third Pod with three labels: `app`, `environment`, and `team`.

### `third-pod.yml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: alpine-pod
  labels:
    app: alpine
    environment: dev
    team: cloud
spec:
  containers:
  - name: alpine
    image: alpine:latest
    command: ["sh", "-c", "echo Hello from Alpine && sleep 3600"]
```

Applied using:

```bash
kubectl apply -f third-pod.yml
```

The Pod started successfully and showed `Running` status.

---

## 5. Imperative vs Declarative Approach

### Imperative

In the imperative approach, I directly tell Kubernetes what command to execute.

Example:

```bash
kubectl run redis-pod --image=redis:latest
```

This created a Redis Pod without manually writing a YAML file.

I inspected the generated resource using:

```bash
kubectl get pod redis-pod -o yaml
```

Kubernetes added additional metadata such as:

- `creationTimestamp`
- `generation`
- `resourceVersion`
- `uid`
- `namespace`
- `status`

### Declarative

In the declarative approach, I define the desired state in a YAML file and apply it.

Example:

```bash
kubectl apply -f nginx-pod.yaml
```

### Difference

| Imperative | Declarative |
|---|---|
| Uses direct commands | Uses YAML manifests |
| Tells Kubernetes what action to perform | Defines the desired state |
| Useful for quick experiments | Better for repeatable infrastructure |
| Example: `kubectl run` | Example: `kubectl apply -f` |

For real-world Kubernetes work, declarative YAML manifests are commonly preferred because they can be stored in Git and version controlled.

---

## 6. Generate YAML with Dry Run

I also used `kubectl` to generate a YAML template without creating the resource:

```bash
kubectl run test-pod --image=nginx --dry-run=client -o yaml
```

The generated YAML can be saved to a file:

```bash
kubectl run test-pod --image=nginx --dry-run=client -o yaml > dry-run-pod.yaml
```

This is useful for quickly creating a basic manifest and then customizing it.

---

## 7. Manifest Validation

I tested both client-side and server-side validation.

### Client-side validation

```bash
kubectl apply -f nginx-pod.yaml --dry-run=client
```

Result:

```text
pod/nginx-pod configured (dry run)
```

### Server-side validation

```bash
kubectl apply -f nginx-pod.yaml --dry-run=server
```

I also intentionally removed the `image` field to test validation.

Kubernetes returned:

```text
The Pod "nginx-pod" is invalid:
spec.containers[0].image: Required value
```

This demonstrated that the container `image` field is required.

---

## 8. Pod Labels

Labels are key-value pairs attached to Kubernetes resources.

I checked labels using:

```bash
kubectl get pods --show-labels
```

Example:

```text
alpine-pod   app=alpine,environment=dev,team=cloud
busybox-pod  app=busybox,environment=dev
nginx-pod    app=nginx
redis-pod    run=redis-pod
```

### Filter Pods by Label

```bash
kubectl get pods -l app=nginx
```

```bash
kubectl get pods -l environment=dev
```

### Add a Label

```bash
kubectl label pod nginx-pod environment=production
```

### Remove a Label

```bash
kubectl label pod nginx-pod environment-
```

This helped me understand how labels can be used to organize and select Kubernetes resources.

---

## 9. Pod Inspection Commands

Commands practiced during this task:

```bash
kubectl get pods
kubectl get pods -o wide
kubectl describe pod nginx-pod
kubectl logs nginx-pod
kubectl exec -it nginx-pod -- /bin/bash
```

These commands are useful for checking Pod status, node/IP information, events, logs, and container access.

---

## 10. Running Pods

During the practical, I successfully created and ran:

- `nginx-pod`
- `busybox-pod`
- `redis-pod`
- `alpine-pod`

The Nginx Pod was tested from inside the container using `curl localhost:80`, and the BusyBox Pod was verified using its logs.


---

## 11. What Happens When a Standalone Pod Is Deleted?

A standalone Pod is not automatically recreated after deletion.

For example:

```bash
kubectl delete pod nginx-pod
```

After deletion, the Pod is gone.

There is no Deployment or other controller managing it, so Kubernetes does not create a replacement Pod.

This is one reason production applications are normally managed using higher-level Kubernetes resources such as **Deployments** rather than directly managing individual Pods.

---
