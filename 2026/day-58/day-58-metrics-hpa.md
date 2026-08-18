# Day 58 – Metrics Server and Horizontal Pod Autoscaler (HPA)


## 1. Kubernetes Cluster

I used a **Kind** Kubernetes cluster for today's practice.

First I checked whether Metrics Server was already running:

```bash
kubectl get pods -n kube-system | grep metrics-server
```

I found that Metrics Server was running, but initially it could not collect metrics because of a kubelet certificate validation issue.

I saw an error related to:

```text
x509: cannot validate certificate
```

Since this is a local Kind cluster, I added:

```text
--kubelet-insecure-tls
```

Then I restarted Metrics Server.

After that:

```bash
kubectl top nodes
```

started working successfully.

> Note: `--kubelet-insecure-tls` is suitable for this local learning environment. It should not be used as a production security solution.

---

## 2. Metrics Server

Metrics Server collects resource usage information from Kubernetes nodes and Pods.

It provides the metrics that commands such as these use:

```bash
kubectl top nodes
kubectl top pods -A
```

### Node usage

My node showed:

```text
CPU:    82m
CPU %:  4%

Memory: 550Mi
Memory %: 7%
```

So at that time my Kind control-plane node was using around **4% CPU and 7% memory**.

---

## 3. Exploring kubectl top

I checked the resource usage of all Pods:

```bash
kubectl top pods -A
```

Then I sorted them by CPU:

```bash
kubectl top pods -A --sort-by=cpu
```

The Pod using the most CPU at that time was:

```text
kube-apiserver-k8s-practice-control-plane
```

It was using around:

```text
23m CPU
204Mi memory
```

### Important difference

I learned that:

```text
kubectl top
```

shows **actual resource usage**.

Whereas:

```yaml
resources:
  requests:
    cpu: 200m
```

defines the amount of CPU requested by the container.

And:

```yaml
resources:
  limits:
    cpu: 500m
```

defines the maximum CPU the container can use.

---

## 4. Creating the PHP-Apache Deployment

Next I created a Deployment using the Kubernetes HPA example image:

```text
registry.k8s.io/hpa-example
```

I configured a CPU request of `200m`.

The Deployment became ready:

```text
php-apache   1/1   1   1
```

The Pod was also running successfully.

---

## 5. Creating the Service

I exposed the Deployment using a ClusterIP Service:

```bash
kubectl expose deployment php-apache --port=80
```

The Service was created successfully:

```text
NAME         TYPE        CLUSTER-IP      PORT(S)
php-apache   ClusterIP   10.96.157.55    80/TCP
```

I then checked the Pod metrics:

```bash
kubectl top pod -l run=php-apache
```

At that time the Pod was using approximately:

```text
CPU: 1m
Memory: 9Mi
```

---

## 6. Creating the HPA

I created the first HPA using:

```bash
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```

I received a deprecation warning for `--cpu-percent`, but the command still worked.

Initially the HPA showed:

```text
cpu: <unknown>/50%
```

After waiting for the metrics to arrive, it changed to:

```text
cpu: 0%/50%
```

I checked:

```bash
kubectl describe hpa php-apache
```

and found:

```text
ScalingActive: True
Reason: ValidMetricFound
```

This confirmed that HPA was successfully receiving CPU metrics.

---

## 7. Generating Load

To test autoscaling, I created a load-generator Pod:

```bash
kubectl run load-generator \
  --image=busybox:1.36 \
  --restart=Never \
  -- /bin/sh -c "while true; do wget -q -O- http://php-apache; done"
```

This continuously sent requests to the PHP-Apache application.

I watched the HPA:

```bash
kubectl get hpa php-apache --watch
```

The CPU usage increased significantly.

At one point I got:

```text
TARGETS: 100%/50%
REPLICAS: 8
```

So Kubernetes scaled the Deployment from the original **1 replica to 8 desired replicas**.

This was the main result I wanted to see from today's practice.

---

## 8. Checking the Scaled Pods

I checked:

```bash
kubectl get pods -l run=php-apache
```

I saw 8 Pods created, but only 4 were running:

```text
READY
1/1
1/1
1/1
1/1
0/1 Pending
0/1 Pending
0/1 Pending
0/1 Pending
```

The Deployment showed:

```text
READY   UP-TO-DATE   AVAILABLE

4/8     8            4
```

At first I was confused because the HPA said 8 replicas but only 4 were available.

Then I checked the events of one Pending Pod:

```bash
kubectl describe pod <pod-name>
```

The important message was:

```text
0/1 nodes are available: 1 Insufficient cpu.
```

### What I learned from this

This showed me an important difference between **HPA and the Kubernetes scheduler**.

HPA decides:

> How many Pods are needed?

The scheduler decides:

> Can these Pods actually be placed on available nodes?

In my case, HPA correctly calculated that more Pods were needed, but my single Kind node did not have enough CPU to run all 8 Pods.

So:

```text
HPA scaling worked
        ↓
8 replicas requested
        ↓
Scheduler tried to place them
        ↓
Only 4 could run
        ↓
Remaining Pods stayed Pending
        ↓
Reason: Insufficient CPU
```

This was one of the most useful things I learned today.

---

## 9. Stopping the Load

I stopped the load generator:

```bash
kubectl delete pod load-generator
```

Even after stopping the load, the HPA did not immediately scale down.

This is because HPA uses a stabilization period for scale-down operations.

My HPA later showed:

```text
cpu: 83%/50%
replicas: 8
```

The scale-down does not happen immediately.

---

# 10. Creating HPA Using YAML

After testing the imperative HPA, I deleted it:

```bash
kubectl delete hpa php-apache
```

Then I created a declarative HPA using the `autoscaling/v2` API.

My configuration included CPU utilization and scaling behavior.


I applied it using:

```bash
kubectl apply -f php-apache-hpa.yml
```

Then checked:

```bash
kubectl describe hpa php-apache
```

The HPA showed:

```text
ScalingActive: True
Reason: ValidMetricFound
```

So the declarative HPA was working correctly.

---

# 11. Understanding HPA Behavior

The `behavior` section controls how quickly Kubernetes scales the application.

### Scale Up

I configured:

```yaml
stabilizationWindowSeconds: 0
```

This means there is no stabilization delay before scaling up.

I also allowed:

```yaml
value: 100
periodSeconds: 15
```

So the HPA can increase replicas quickly when the application needs more capacity.

### Scale Down

I configured:

```yaml
stabilizationWindowSeconds: 300
```

This means Kubernetes uses a **5-minute stabilization window** for scale-down.

This helps prevent the application from constantly scaling up and down when traffic changes.

---

# 12. autoscaling/v1 vs autoscaling/v2

I also learned the difference between the two HPA API versions.

### autoscaling/v1

Mainly supports CPU-based autoscaling.

It is simpler and useful for basic HPA configurations.

### autoscaling/v2

Provides more advanced configuration.

It supports:

* CPU metrics
* Memory metrics
* Multiple metrics
* Scaling behavior
* Scale-up policies
* Scale-down policies
* Stabilization windows

For more advanced HPA configurations, `autoscaling/v2` is the better choice.

---

# 13. HPA Formula

The basic idea behind HPA scaling can be represented as:

```text
desiredReplicas =
ceil(currentReplicas × currentUsage / targetUsage)
```

For example, if:

```text
Current replicas = 2
Current CPU = 100%
Target CPU = 50%
```

Then:

```text
2 × (100 / 50) = 4
```

So HPA can recommend:

```text
4 replicas
```

This allows Kubernetes to automatically adjust the number of Pods according to resource usage.

---






