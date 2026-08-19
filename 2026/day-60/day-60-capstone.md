# Day 60 – Kubernetes Capstone: WordPress + MySQL

## Overview

For Day 60 of #90DaysOfDevOps, I combined the Kubernetes concepts I learned over the previous days into one complete application deployment.

I deployed a WordPress + MySQL application in a dedicated `capstone` namespace on a Kubernetes cluster running with Kind on an AWS EC2 instance.

The deployment included Namespaces, Secrets, ConfigMaps, StatefulSets, PersistentVolumeClaims, Headless Services, Deployments, NodePort Services, resource requests and limits, health probes, HPA, and Helm.

---

## Environment

* Cloud: AWS EC2
* OS: Ubuntu
* Kubernetes: Kind
* Kubernetes version: v1.36.1
* Application: WordPress
* Database: MySQL 8.0
* Namespace: `capstone`

---

## Architecture

```text
                         capstone namespace
                                |
                +---------------+---------------+
                |                               |
          WordPress                         MySQL
         Deployment                       StatefulSet
           2 Pods                            1 Pod
                |                               |
                |                         mysql-0
                |                               |
                |                         1Gi PVC
                |                               |
                |                         wordpress DB
                |
        WordPress Service
           NodePort
          80:30080
                |
                |
          WordPress site

WordPress configuration
        |
        +-- ConfigMap
        |     |
        |     +-- WORDPRESS_DB_HOST
        |     +-- WORDPRESS_DB_NAME
        |
        +-- Secret
              |
              +-- DB username
              +-- DB password

WordPress Deployment
        |
        +-- Resource requests/limits
        +-- Liveness probe
        +-- Readiness probe
        |
        +-- HPA
              |
              +-- Minimum: 2
              +-- Maximum: 10
              +-- CPU target: 50%
```

---

## Resources Created

### Namespace

Created a dedicated namespace:

```bash
kubectl create namespace capstone
```

The current Kubernetes context was configured to use `capstone` as the default namespace.

### MySQL Secret

Created `mysql-secret` using `stringData`.

It contains:

* `MYSQL_ROOT_PASSWORD`
* `MYSQL_DATABASE`
* `MYSQL_USER`
* `MYSQL_PASSWORD`

### MySQL StatefulSet

Created a MySQL 8.0 StatefulSet with:

* 1 replica
* Stable Pod identity: `mysql-0`
* Resource requests:

  * CPU: `250m`
  * Memory: `512Mi`
* Resource limits:

  * CPU: `500m`
  * Memory: `1Gi`
* Persistent storage mounted at `/var/lib/mysql`

### MySQL Headless Service

Created a headless Service:

```text
mysql
ClusterIP: None
Port: 3306
```

This provides stable DNS for the MySQL StatefulSet.

The WordPress database host was configured as:

```text
mysql-0.mysql.capstone.svc.cluster.local:3306
```

### PersistentVolumeClaim

The MySQL StatefulSet created:

```text
mysql-storage-mysql-0
```

with:

```text
Capacity: 1Gi
Access Mode: RWO
StorageClass: standard
Status: Bound
```

### WordPress ConfigMap

Created `wordpress-config` containing:

```text
WORDPRESS_DB_HOST=mysql-0.mysql.capstone.svc.cluster.local:3306
WORDPRESS_DB_NAME=wordpress
```

### WordPress Deployment

Created a Deployment with:

* 2 replicas
* Image: `wordpress:latest`
* CPU request: `100m`
* Memory request: `256Mi`
* CPU limit: `500m`
* Memory limit: `512Mi`

Database username and password were injected using `secretKeyRef`.

### Health Probes

Configured both:

* Liveness probe
* Readiness probe

Both check:

```text
/wp-login.php
```

on port 80.

### WordPress Service

Created a NodePort Service:

```text
Port: 80
NodePort: 30080
```

For browser access from the EC2 environment, I also used:

```bash
kubectl port-forward --address 0.0.0.0 svc/wordpress 8080:80
```

The WordPress site was accessed through the EC2 public IP on port 8080.

---

## Verification Results

### MySQL Database

Verified the database from inside the MySQL Pod:

```bash
kubectl exec -it mysql-0 -- mysql -u wordpress -p<password> -e "SHOW DATABASES;"
```

The `wordpress` database was successfully present.

### WordPress Pods

Final state:

```text
wordpress-564dcf79fd-fr85n   1/1   Running
wordpress-564dcf79fd-pbhtt   1/1   Running
```

The Deployment showed:

```text
2/2 desired
2/2 updated
2/2 available
```

### MySQL Pod

Final state:

```text
mysql-0   1/1   Running
```

### Persistent Storage

Final PVC:

```text
mysql-storage-mysql-0   Bound   1Gi   RWO   standard
```

---

## Self-Healing Test

I manually deleted one WordPress Pod:

```bash
kubectl delete pod wordpress-564dcf79fd-6ccw4
```

Kubernetes automatically created a replacement Pod.

The replacement changed from:

```text
0/1 Running
```

to:

```text
1/1 Running
```

The Deployment maintained the desired replica count of 2.

### Result

**WordPress self-healing: PASSED**

---

## MySQL Self-Healing Test

I manually deleted the MySQL Pod:

```bash
kubectl delete pod mysql-0
```

The StatefulSet automatically recreated:

```text
mysql-0
```

The Pod returned to:

```text
1/1 Running
```

### Result

**MySQL self-healing: PASSED**

---

## Persistence Test

After deleting and recreating the MySQL Pod, I checked the databases again.

The `wordpress` database was still present.

I also refreshed the WordPress application and verified that the WordPress blog post created during the setup was still there.

This proved that the MySQL data was stored on the PersistentVolumeClaim rather than being tied only to the lifetime of the Pod.

### Result

**Data persistence: PASSED**

---

## HPA Configuration

Metrics Server was installed and verified using:

```bash
kubectl top nodes
kubectl top pods
```

CPU and memory metrics were successfully returned.

The WordPress HPA was configured with:

```text
Minimum replicas: 2
Maximum replicas: 10
CPU target: 50%
```

Final HPA output:

```text
wordpress-hpa   Deployment/wordpress   cpu: 3%/50%   2   10   2
```

The HPA reported:

```text
ScalingActive: True
Reason: ValidMetricFound
```

This confirmed that Kubernetes was successfully calculating CPU utilization.

### Result

**HPA: PASSED**

---

## Helm Comparison

As a bonus, I installed WordPress using the Bitnami Helm chart in a separate namespace.

Command used:

```bash
helm install wp-helm bitnami/wordpress --namespace helm-capstone
```

The Helm release was successfully deployed.

The chart created resources including:

* WordPress Deployment
* MariaDB StatefulSet
* WordPress Service
* MariaDB Service
* MariaDB Headless Service
* PersistentVolumeClaims
* Secrets

The Bitnami chart used MariaDB rather than the MySQL 8.0 deployment used in my manual capstone.

After the comparison, the Helm deployment was removed:

```bash
helm uninstall wp-helm -n helm-capstone
kubectl delete namespace helm-capstone
```

### Manual Kubernetes vs Helm

| Manual Kubernetes                        | Helm                                  |
| ---------------------------------------- | ------------------------------------- |
| More control over individual resources   | Faster application deployment         |
| Better for learning Kubernetes internals | Reusable charts                       |
| Explicit YAML configuration              | Values-based configuration            |
| More files to maintain                   | Easier upgrades and rollbacks         |
| Easier to understand each component      | Abstracts many implementation details |

### My takeaway

The manual deployment gave me a much deeper understanding of how WordPress, MySQL, Services, storage, configuration, and scaling connect together.

Helm showed me how the same application can be packaged and deployed much faster using a reusable chart.

---

## Kubernetes Concepts Used

| Concept                     | Learned |
| --------------------------- | ------- |
| Namespace                   | Day 52  |
| Services                    | Day 53  |
| ConfigMaps                  | Day 52  |
| Secrets                     | Day 52  |
| PersistentVolume / PVC      | Day 55  |
| StatefulSets                | Day 56  |
| Resource Requests & Limits  | Day 57  |
| Liveness & Readiness Probes | Day 57  |
| HPA                         | Day 58  |
| Helm                        | Day 59  |
| Deployment                  | Day 52  |
| Headless Service            | Day 56  |
| NodePort                    | Day 53  |

---

---

## What Was Hardest?

The most challenging part was connecting all the Kubernetes concepts together rather than learning each resource individually.

Earlier, I worked with Deployments, StatefulSets, Services, storage, probes, resource limits, HPA, and Helm separately.

In this capstone, I had to understand how they interact.

The networking between WordPress and MySQL was also important because WordPress had to use the correct StatefulSet DNS address.

---

## What Clicked?

The biggest thing that clicked for me was understanding the difference between stateless and stateful workloads.

WordPress was managed by a Deployment because the Pods could be replaced.

MySQL was managed by a StatefulSet because it needed stable identity and persistent storage.

The self-healing test made this much clearer because deleting both Pods produced different Kubernetes behaviors while the database data remained available.

---




