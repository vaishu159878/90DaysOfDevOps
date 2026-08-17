# Day 55 – Persistent Volumes (PV) and Persistent Volume Claims (PVC)

## 1. Why Persistent Storage Is Needed

Pods are temporary in Kubernetes.

If a Pod is deleted, data stored only inside the Pod can be lost. This is fine for temporary application data, but it is a problem for applications such as:

* Databases
* File uploads
* Application data
* Logs that need to survive Pod restarts
* Stateful applications

Kubernetes provides persistent storage resources so that the storage lifecycle can be separated from the Pod lifecycle.

---

# 2. Task 1 – Demonstrating Ephemeral Storage

I created a Pod using an `emptyDir` volume.

The Pod wrote a timestamped message to:

```text
/data/message.txt
```

I checked the file using:

```bash
kubectl exec ephemeral-pod -- cat /data/message.txt
```

The first timestamp was:

```text
Created at Mon Aug 17 14:09:17 UTC 2026
```

After deleting and recreating the Pod, the timestamp changed:

```text
Created at Mon Aug 17 14:10:16 UTC 2026
```

### Result

The timestamp was different, which proved that the previous data was lost when the Pod was deleted.

```text
emptyDir

Pod
 ↓
Data
 ↓
Pod deleted
 ↓
Data lost
```

`emptyDir` is therefore useful for temporary storage, but it is not persistent storage.

---

# 3. Task 2 – Creating a PersistentVolume

I created a static PersistentVolume with:

* Capacity: `1Gi`
* Access mode: `ReadWriteOnce`
* Reclaim policy: `Retain`
* Storage type: `hostPath`

The PV used:

```text
/tmp/k8s-pv-data
```

I checked the PV using:

```bash
kubectl get pv
```

The initial status was:

```text
manual-pv   1Gi   RWO   Retain   Available
```

### PV Status

A PV can move through different states:

```text
Available → Bound → Released
```

* **Available** – PV is available to be claimed.
* **Bound** – PV is bound to a PVC.
* **Released** – PVC was deleted but the PV was retained.

> `hostPath` is useful for learning and local testing, but it is not normally suitable for production storage.

---

# 4. Task 3 – Creating a PersistentVolumeClaim

I created a PVC requesting:

```text
500Mi
ReadWriteOnce
```

Initially, my PVC was trying to use the default StorageClass.

Because my manually created PV did not have a StorageClass, Kubernetes did not bind the PVC to the manual PV.

I fixed this by explicitly setting:

```yaml
storageClassName: ""
```

This disabled dynamic provisioning for the PVC.

After applying the corrected PVC, I checked:

```bash
kubectl get pvc
kubectl get pv
```

The result showed:

```text
manual-pvc   Bound   manual-pv   1Gi   RWO
```

and:

```text
manual-pv   1Gi   RWO   Retain   Bound   default/manual-pvc
```

This confirmed that the PVC was correctly bound to the manually created PV.

### Important lesson

The default StorageClass can affect PVC binding.

Using:

```yaml
storageClassName: ""
```

allows a PVC to request an existing PV without dynamically provisioning another one.

---

# 5. Task 4 – Using the PVC in a Pod

I created a Pod and mounted the PVC at:

```text
/data
```

The Pod used:

```yaml
persistentVolumeClaim:
  claimName: manual-pvc
```

I wrote data into:

```text
/data/message.txt
```

The first Pod wrote:

```text
Pod created at Mon Aug 17 14:15:20 UTC 2026
```

After deleting and recreating the Pod, the existing data was still available.

The file contained:

```text
Pod created at Mon Aug 17 14:15:20 UTC 2026
Pod created at Mon Aug 17 14:16:30 UTC 2026
Second Pod: Mon Aug 17 14:21:45 UTC 2026
```

### Result

The data survived Pod deletion.

```text
Pod 1
 ↓
PVC
 ↓
PV
 ↓
Data

Pod 1 deleted
 ↓
Pod 2
 ↓
Same PVC
 ↓
Same PV
 ↓
Data still available
```

This was the main difference I learned between `emptyDir` and persistent storage.

---

# 6. Task 5 – StorageClasses

I checked the StorageClass in my cluster using:

```bash
kubectl get storageclass
```

My cluster had:

```text
standard (default)
```

I also checked:

```bash
kubectl describe storageclass standard
```

### StorageClass details

| Property            | Value                   |
| ------------------- | ----------------------- |
| Name                | `standard`              |
| Default             | Yes                     |
| Provisioner         | `rancher.io/local-path` |
| Reclaim Policy      | `Delete`                |
| Volume Binding Mode | `WaitForFirstConsumer`  |

The StorageClass is responsible for dynamic storage provisioning.

---

# 7. Task 6 – Dynamic Provisioning

I created another PVC using:

```yaml
storageClassName: standard
```

After creating the PVC, it initially showed:

```text
dynamic-pvc   Pending
```

This happened because my StorageClass uses:

```text
WaitForFirstConsumer
```

The StorageClass waits until a Pod actually uses the PVC.

I then created a Pod using `dynamic-pvc`.

After the Pod was created, I checked:

```bash
kubectl get pvc
kubectl get pv
```

The PVC became:

```text
dynamic-pvc   Bound
```

Kubernetes automatically created a PV:

```text
pvc-376d7898-bc62-42fb-8bf6-b2c246a3a112
```

The automatically created PV had:

```text
Capacity: 500Mi
Access Mode: RWO
Reclaim Policy: Delete
StorageClass: standard
Status: Bound
```

I also wrote data to the dynamically provisioned volume:

```text
Dynamic PV data: Mon Aug 17 14:25:15 UTC 2026
```

This proved that dynamic provisioning was working.

---

# 8. Static vs Dynamic Provisioning

## Static Provisioning

In static provisioning, the PV is created manually.

```text
Administrator
     ↓
PersistentVolume
     ↓
PersistentVolumeClaim
     ↓
Pod
```

My manually created volume was:

```text
manual-pv
```

---

## Dynamic Provisioning

In dynamic provisioning, the user creates a PVC and the StorageClass automatically creates the PV.

```text
User
 ↓
PVC
 ↓
StorageClass
 ↓
PV automatically created
 ↓
Pod
```

My dynamically created PV had a name similar to:

```text
pvc-376d7898-bc62-42fb-8bf6-b2c246a3a112
```

---

# 9. Access Modes

Kubernetes supports different access modes.

## ReadWriteOnce (RWO)

```text
Read + Write
Single node
```

The volume can be mounted as read-write by workloads on a single node.

## ReadOnlyMany (ROX)

```text
Read only
Multiple nodes
```

Multiple nodes can mount the volume as read-only.

## ReadWriteMany (RWX)

```text
Read + Write
Multiple nodes
```

Multiple nodes can mount the volume with read-write access.

The actual access modes available depend on the underlying storage system.

---

# 10. Reclaim Policies

I also tested how reclaim policies behave when a PVC is deleted.

## Retain

My manual PV used:

```text
persistentVolumeReclaimPolicy: Retain
```

After deleting `manual-pvc`, the PV remained:

```text
manual-pv   1Gi   RWO   Retain   Released
```

So the storage was retained instead of being automatically deleted.

---

## Delete

The dynamically created PV used:

```text
Reclaim Policy: Delete
```

After deleting `dynamic-pvc`, Kubernetes automatically removed the dynamically created PV.

### Difference

```text
Retain:

PVC deleted
    ↓
PV remains
    ↓
Released


Delete:

PVC deleted
    ↓
PV automatically deleted
```

---

# 11. PV vs PVC

| PersistentVolume (PV)                | PersistentVolumeClaim (PVC)        |
| ------------------------------------ | ---------------------------------- |
| Provides actual storage              | Requests storage                   |
| Cluster-scoped                       | Namespace-scoped                   |
| Can be manually created              | Created by users/applications      |
| Can be dynamically provisioned       | Can trigger dynamic provisioning   |
| Has capacity and access modes        | Requests capacity and access modes |
| Can be Available, Bound, or Released | Can be Pending or Bound            |

A simple way to remember:

```text
PV  = Storage provided
PVC = Storage requested
```

---

