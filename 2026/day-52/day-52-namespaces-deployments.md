# Day 52 -- Kubernetes Namespaces and Deployments

------------------------------------------------------------------------

## 1. Exploring Kubernetes Namespaces

I started by checking the namespaces available in my cluster:

``` bash
kubectl get namespaces
```

I found these namespaces:

-   `default`
-   `kube-node-lease`
-   `kube-public`
-   `kube-system`
-   `local-path-storage`

Then I checked the Pods inside `kube-system`:

``` bash
kubectl get pods -n kube-system
```

There were **8 Pods running**.

These Pods are related to Kubernetes control-plane and cluster
components, so I did not modify them.

------------------------------------------------------------------------

## 2. Creating Custom Namespaces

I created two namespaces for my practice:

``` bash
kubectl create namespace dev
kubectl create namespace staging
```

Both namespaces were created successfully.

Then I created an Nginx Pod in each namespace:

``` bash
kubectl run nginx-dev --image=nginx:latest -n dev
kubectl run nginx-staging --image=nginx:latest -n staging
```

When I ran:

``` bash
kubectl get pods
```

I got:

``` text
No resources found in default namespace.
```

This helped me understand that `kubectl get pods` only checks the
current/default namespace.

To see Pods from all namespaces, I used:

``` bash
kubectl get pods -A
```

This showed the Pods from `dev`, `staging`, `kube-system`, and other
namespaces.

------------------------------------------------------------------------

## 3. Creating a Deployment

I created an Nginx Deployment using `nginx-deployment.yaml`.

### Deployment Manifest

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.24
        ports:
        - containerPort: 80
```

I applied it using:

``` bash
kubectl apply -f nginx-deployment.yaml
```

Then I checked the Deployment:

``` bash
kubectl get deployments -n dev
```

The result was:

``` text
NAME               READY   UP-TO-DATE   AVAILABLE
nginx-deployment   3/3     3            3
```
------------------------------------------------------------------------

## 4. Self-Healing

One important feature I tested was self-healing.

I first checked the Pods:

``` bash
kubectl get pods -n dev
```

Then I deleted one Pod managed by the Deployment:

``` bash
kubectl delete pod <pod-name> -n dev
```

After deleting it, I checked the Pods again:

``` bash
kubectl get pods -n dev
```

Kubernetes automatically created a replacement Pod.

The new Pod had a **different name** from the deleted Pod.

This showed me that the Deployment maintains the desired number of
replicas through its ReplicaSet.

------------------------------------------------------------------------

## 5. Scaling the Deployment

I scaled the Deployment from 3 replicas to 5:

``` bash
kubectl scale deployment nginx-deployment --replicas=5 -n dev
```

Kubernetes created additional Pods until 5 replicas were running.

Then I scaled it down to 2:

``` bash
kubectl scale deployment nginx-deployment --replicas=2 -n dev
```

The extra Pods were terminated, leaving 2 running Pods.

This helped me understand how Kubernetes maintains the desired replica
count.

------------------------------------------------------------------------

## 6. Rolling Update

I updated the Nginx image from version `1.24` to `1.25`:

``` bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.25 -n dev
```

Then I checked the rollout:

``` bash
kubectl rollout status deployment/nginx-deployment -n dev
```

The rollout completed successfully:

``` text
deployment "nginx-deployment" successfully rolled out
```

Kubernetes created the new version of the Pods and replaced the old Pods
as part of the rolling update.

------------------------------------------------------------------------

## 7. Rollout History

I checked the Deployment history:

``` bash
kubectl rollout history deployment/nginx-deployment -n dev
```

I could see two revisions:

``` text
REVISION
1
2
```

------------------------------------------------------------------------

## 8. Rollback

I rolled the Deployment back to the previous version:

``` bash
kubectl rollout undo deployment/nginx-deployment -n dev
```

Then I checked the rollout status:

``` bash
kubectl rollout status deployment/nginx-deployment -n dev
```

Finally, I verified the image:

``` bash
kubectl describe deployment nginx-deployment -n dev | grep Image
```

The result showed:

``` text
Image: nginx:1.24
```

So the rollback was successful.

------------------------------------------------------------------------

## 9. Final Verification

I checked the Deployment:

``` bash
kubectl get deployments -n dev
```

Result:

``` text
NAME               READY   UP-TO-DATE   AVAILABLE
nginx-deployment   2/2     2            2
```

I also checked Pods in `dev` and `staging`:

``` bash
kubectl get pods -n dev
kubectl get pods -n staging
```

And finally:

``` bash
kubectl get pods -A
```

This showed resources running across multiple namespaces.

------------------------------------------------------------------------

