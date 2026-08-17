# Day 56 – Kubernetes StatefulSets

## Objective

Today I learned how Kubernetes handles **stateful applications** using StatefulSets.

I already understood that Deployments are useful for stateless applications, but I wanted to see what happens when an application needs:

- Stable Pod names
- Ordered Pod creation and deletion
- Stable network identity
- Persistent storage for each Pod

For this practical, I used **nginx** to understand how StatefulSets work.

---

## 1. Deployment vs StatefulSet

First, I created a Deployment with 3 nginx replicas.

The Pods received generated names such as:

```text
web-deployment-b6b6598c4-hlnd6
web-deployment-b6b6598c4-mcpw9
web-deployment-b6b6598c4-zm258
```

I deleted one Pod and Kubernetes created a replacement with a different name.

This showed me that Deployment Pods are treated as interchangeable.

For stateless applications this is usually fine, but stateful applications such as databases may need each instance to keep its own identity.

### Deployment vs StatefulSet

| Feature | Deployment | StatefulSet |
|---|---|---|
| Pod names | Generated/random-looking | Stable and ordered |
| Example | `app-xyz-abc` | `app-0`, `app-1`, `app-2` |
| Pod identity | Interchangeable | Stable identity |
| Startup | Pods can start independently | Ordered by default |
| Storage | Usually shared/application-dependent | Separate PVC per Pod |
| Network identity | No stable Pod hostname | Stable Pod DNS |
| Good for | Stateless applications | Stateful applications |

---

## 2. Headless Service

Before creating the StatefulSet, I created a Headless Service.

The important configuration was:

```yaml
clusterIP: None
```

I verified it using:

```bash
kubectl get svc
```

The result showed:

```text
web-headless   ClusterIP   None
```

A Headless Service does not provide one virtual ClusterIP for load balancing. Instead, it allows Kubernetes DNS to provide records for individual Pods.

This is useful with StatefulSets because each Pod can have its own stable DNS name.


---

## 3. Creating the StatefulSet

I created a StatefulSet called `web` with 3 nginx replicas.

The StatefulSet used:

```yaml
serviceName: web-headless
replicas: 3
```

and a `volumeClaimTemplates` section requesting:

```text
100Mi
ReadWriteOnce
```

After applying it, Kubernetes created the Pods in order:

```text
web-0
web-1
web-2
```

I also verified the StatefulSet:

```bash
kubectl get sts
```

and got:

```text
NAME   READY
web    3/3
```

### PVCs

Kubernetes automatically created one PVC for each Pod:

```text
web-data-web-0
web-data-web-1
web-data-web-2
```

Each PVC had:

```text
100Mi
RWO
Bound
```

This helped me understand that `volumeClaimTemplates` creates a separate storage claim for each StatefulSet replica.


---

## 4. Stable Network Identity

I created a temporary BusyBox Pod and used `nslookup` to test the StatefulSet DNS records.

The DNS pattern is:

```text
<pod-name>.<service-name>.<namespace>.svc.cluster.local
```

For my setup:

```text
web-0.web-headless.default.svc.cluster.local
web-1.web-headless.default.svc.cluster.local
web-2.web-headless.default.svc.cluster.local
```

The DNS lookups returned:

```text
web-0 → 10.244.0.10
web-1 → 10.244.0.12
web-2 → 10.244.0.14
```

I then checked the Pod IPs:

```bash
kubectl get pods -o wide
```

and confirmed that the DNS results matched the actual Pod IPs.

This showed me that each StatefulSet Pod gets a stable DNS identity.

---

## 5. Persistent Storage Test

This was the most important part of the practical.

I wrote custom data into `web-0`:

```bash
kubectl exec web-0 -- sh -c "echo 'Data from web-0' > /usr/share/nginx/html/index.html"
```

Then I verified it:

```bash
kubectl exec web-0 -- cat /usr/share/nginx/html/index.html
```

Output:

```text
Data from web-0
```

Next, I deleted the Pod:

```bash
kubectl delete pod web-0
```

Kubernetes recreated the Pod with the same name:

```text
web-0
```

After the replacement Pod became ready, I checked the file again:

```bash
kubectl exec web-0 -- cat /usr/share/nginx/html/index.html
```

The data was still:

```text
Data from web-0
```

This proved that the new `web-0` reattached to its existing PVC:

```text
web-data-web-0
```

So the data survived Pod deletion.

---

## 6. Ordered Scaling

I scaled the StatefulSet from 3 replicas to 5:

```bash
kubectl scale statefulset web --replicas=5
```

Kubernetes created the new Pods in order:

```text
web-3
web-4
```

The StatefulSet then had:

```text
web-0
web-1
web-2
web-3
web-4
```

Five PVCs were also present:

```text
web-data-web-0
web-data-web-1
web-data-web-2
web-data-web-3
web-data-web-4
```

I then scaled it back down:

```bash
kubectl scale statefulset web --replicas=3
```

The Pods were removed in reverse order:

```text
web-4
web-3
```

The remaining Pods were:

```text
web-0
web-1
web-2
```

However, all five PVCs were still present.

So after scaling down from 5 Pods to 3 Pods:

**5 PVCs remained.**


---

## 7. Cleanup

I deleted the StatefulSet:

```bash
kubectl delete statefulset web
```

Then I deleted the Headless Service:

```bash
kubectl delete service web-headless
```

After deleting the StatefulSet, the PVCs were still present.

This demonstrated that deleting a StatefulSet does not automatically delete its PVCs.

I then manually deleted the PVCs:

```bash
kubectl delete pvc web-data-web-0 web-data-web-1 web-data-web-2 web-data-web-3 web-data-web-4
```

Finally, I removed the temporary DNS testing Pod:

```bash
kubectl delete pod dns-test
```

The final checks showed no Pods, StatefulSets, or PVCs remaining in the default namespace.

---

## 8. What I Learned

The biggest thing I understood today is that **StatefulSets are about identity and state**.

A Deployment can replace:

```text
app-abc123
```

with:

```text
app-xyz456
```

and the application usually doesn't care.

A StatefulSet maintains identities such as:

```text
web-0
web-1
web-2
```

Each Pod also gets its own PVC and stable DNS identity.
---



