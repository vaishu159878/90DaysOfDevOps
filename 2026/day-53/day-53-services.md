# Day 53 – Kubernetes Services

## Objective

Today I learned how Kubernetes Services provide a stable way to access Pods.

Pods can be recreated at any time, so their IP addresses are not permanent. A Service solves this problem by giving the application a stable IP and DNS name and by forwarding traffic to the Pods selected by its labels.

For this task, I created an Nginx Deployment with 3 replicas and exposed it using:

- ClusterIP
- NodePort
- LoadBalancer

I also tested Service communication from inside the cluster and verified Kubernetes DNS.

---

## 1. Creating the Kubernetes Cluster

I created a local Kubernetes cluster using Kind:

```bash
kind create cluster --name k8s-practice
```

The cluster was created successfully and the kubectl context was automatically configured.

---

## 2. Deploying the Application

I created a Deployment called `web-app` with 3 Nginx replicas.

```bash
kubectl apply -f app-deployment.yaml
kubectl get pods -o wide
```

All 3 Pods were running successfully.

The Pod IPs were:

```text
10.244.0.5
10.244.0.7
10.244.0.6
```

These IPs belong to individual Pods and can change if a Pod is recreated. This is one of the main reasons we use Services.

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

---

## 3. ClusterIP Service

ClusterIP is the default Kubernetes Service type. It is mainly used for communication inside the cluster.

I created:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-clusterip
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
```

I applied it using:

```bash
kubectl apply -f clusterip-service.yaml
kubectl get services
```

The Service received this ClusterIP:

```text
10.96.22.39
```

The important part here is the selector:

```yaml
selector:
  app: web-app
```

It matches the `app: web-app` label on my Pods, so the Service knows which Pods should receive traffic.

---

## 4. Testing Pod-to-Service Communication

To test the ClusterIP Service from inside the cluster, I created a temporary BusyBox Pod:

```bash
kubectl run test-client --image=busybox:latest --rm -it --restart=Never -- sh
```

Inside the Pod, I tested:

```bash
wget -qO- http://web-app-clusterip
```

The request returned the Nginx welcome page.

So the Service was successfully routing traffic to the application Pods.

---

## 5. Kubernetes DNS

Kubernetes automatically creates DNS records for Services.

The general format is:

```text
<service-name>.<namespace>.svc.cluster.local
```

For my Service:

```text
web-app-clusterip.default.svc.cluster.local
```

I tested the short Service name:

```bash
wget -qO- http://web-app-clusterip
```

and the full DNS name:

```bash
wget -qO- http://web-app-clusterip.default.svc.cluster.local
```

Both returned the Nginx welcome page.

I also checked the DNS entry:

```bash
nslookup web-app-clusterip
```

The Service resolved to:

```text
10.96.22.39
```

which matched the ClusterIP shown by Kubernetes.

I also noticed some `NXDOMAIN` messages from `nslookup` for additional search-domain variations. The actual Service name and full DNS name resolved correctly, so the Service DNS was working.

---

## 6. NodePort Service

Next, I created a NodePort Service.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-nodeport
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

I applied it with:

```bash
kubectl apply -f nodeport-service.yaml
kubectl get services
```

The Service was created with:

```text
80:30080/TCP
```

Here:

- `80` = Service port
- `80` = Pod targetPort
- `30080` = NodePort

I then tested it using the Kind node IP:

```bash
curl http://172.18.0.2:30080
```

The request returned the Nginx welcome page.

This confirmed that external traffic could reach the application through the NodePort.

---

## 7. LoadBalancer Service

Finally, I created a LoadBalancer Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
```

I initially had a YAML formatting error while creating this file:

```text
error parsing loadbalancer-service.yaml:
yaml: line 12: found character that cannot start any token
```

I fixed the YAML and applied it again successfully:

```bash
kubectl apply -f loadbalancer-service.yaml
```

The Service was then created successfully.

Because I am using a local Kind cluster instead of a cloud Kubernetes cluster with a LoadBalancer controller, the external IP remains:

```text
<pending>
```

This is expected.

The LoadBalancer Service received:

```text
ClusterIP: 10.96.55.153
NodePort: 30225
```

---

## 8. Services Side by Side

I checked all Services using:

```bash
kubectl get services -o wide
```

All three application Services use:

```text
app=web-app
```

as their selector.

### Comparison

| Service Type | Access | Main Use |
|---|---|---|
| ClusterIP | Inside cluster | Internal application communication |
| NodePort | Node IP + port | Development/testing and direct external access |
| LoadBalancer | External load balancer | External traffic in cloud environments |

---

## 9. Checking the LoadBalancer Service

I used:

```bash
kubectl describe service web-app-loadbalancer
```

This helped me understand that a LoadBalancer Service also gets a ClusterIP and, in this setup, a NodePort.

The endpoints show the actual Pod IPs receiving traffic.

---

## 10. What I Learned

The biggest thing I understood today is that I should not directly depend on Pod IP addresses for application communication.

Pods can be recreated and their IP addresses can change.


The Service provides a stable endpoint and selects the correct Pods using labels.

I also understood the difference between:

- `port` – port exposed by the Service
- `targetPort` – port where the application is running inside the Pod
- `nodePort` – port exposed on the Kubernetes node

---

## 13. Cleanup

After completing the task, the resources can be removed using:

```bash
kubectl delete -f app-deployment.yaml
kubectl delete -f clusterip-service.yaml
kubectl delete -f nodeport-service.yaml
kubectl delete -f loadbalancer-service.yaml
```

Then verify:

```bash
kubectl get pods
kubectl get services
```

The built-in `kubernetes` Service should remain.

---

