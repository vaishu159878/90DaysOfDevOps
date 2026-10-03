# Helm — Chart Basics & Release Management

## Overview

This project demonstrates the fundamentals of **Helm**, the package manager for Kubernetes.

I used Helm to deploy and manage MySQL on a local Kubernetes cluster running with Kind.

---

## Environment

- Kubernetes: Kind
- Cluster: `tws-cluster`
- Helm: v3.22.0
- Chart: Bitnami MySQL
- Chart Version: `14.0.3`
- MySQL App Version: `9.4.0`

---

## Helm Architecture

```text
Helm
 │
 ├── Repository
 │      └── Bitnami
 │
 ├── Chart
 │      └── MySQL
 │
 ├── Values
 │      └── Configuration
 │
 └── Release
        └── bankapp-mysql
```

### Important Concepts

| Concept | Meaning |
|---|---|
| Chart | Package containing Kubernetes templates |
| Release | Installed instance of a Helm chart |
| Repository | Location where Helm charts are stored |
| Values | Configuration used to customize a chart |
| Revision | Version of a Helm release during its lifecycle |

---

## 1. Add Helm Repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

Search for MySQL:

```bash
helm search repo bitnami/mysql
```

---

## 2. Install MySQL

The MySQL application was deployed using the Bitnami Helm chart.

The deployment included:

- MySQL database
- Persistent storage
- Kubernetes StatefulSet
- Service
- Secret
- ConfigMap

The environment required a legacy MySQL image because the chart's referenced image was unavailable for the required tag.

The working image configuration was:

```text
bitnamilegacy/mysql:9.4.0-debian-12-r1
```

The chart also required:

```text
global.security.allowInsecureImages=true
```

for this substituted-image learning setup.

---

## 3. Verify the MySQL Release

```bash
helm list
```

Example:

```text
NAME            STATUS      CHART
bankapp-mysql   deployed    mysql-14.0.3
```

Check Kubernetes:

```bash
kubectl get pods
```

Expected:

```text
bankapp-mysql-0   1/1   Running
```

---

## 4. Verify the Database

The MySQL database was verified from inside the running pod.

```bash
kubectl exec -it bankapp-mysql-0 -- \
  mysql -uroot -p \
  -e "SHOW DATABASES;"
```

The database created for the application was:

```text
bankappdb
```

> Never commit database passwords or other credentials to a public Git repository.

---

## 5. Custom Values

Helm allows configuration to be separated from the chart templates.

Example configuration:

```yaml
auth:
  database: bankappdb

primary:
  resources:
    limits:
      cpu: 500m
      memory: 512Mi
    requests:
      cpu: 250m
      memory: 256Mi

  persistence:
    size: 5Gi
```

This makes the deployment easier to configure without modifying the chart itself.

---

## 6. Deploy a Second Release

The same chart can be installed multiple times with different release names.

Example:

```bash
helm install bankapp-mysql-v2 bitnami/mysql \
  -f mysql-values.yaml
```

The second release initially encountered an image-pull problem with the MySQL metrics sidecar.

The failing image was:

```text
bitnami/mysqld-exporter:0.17.2-debian-12-r16
```

For this learning environment, metrics were disabled:

```bash
--set metrics.enabled=false
```

After recreating the pod, the release became healthy.

This demonstrated that multiple independent releases can be created from the same chart.

---

## 7. Uninstall a Release

The temporary second release was removed using:

```bash
helm uninstall bankapp-mysql-v2
```

Verification:

```bash
helm list
kubectl get pods
```

The main `bankapp-mysql` release remained deployed.

---

## 8. Helm Upgrade

A safe configuration change was used to practice upgrading a release.

```bash
helm upgrade bankapp-mysql bitnami/mysql \
  -f mysql-values.yaml \
  --set global.security.allowInsecureImages=true \
  --set image.repository=bitnamilegacy/mysql \
  --set image.tag=9.4.0-debian-12-r1 \
  --set metrics.enabled=false \
  --set primary.resources.requests.cpu=300m
```

The release was upgraded successfully.

---

## 9. StatefulSet Upgrade Failure

I also attempted to change the persistent volume size from:

```text
5Gi → 6Gi
```

The Helm upgrade failed because the StatefulSet specification contains fields that Kubernetes does not allow to be modified through a normal StatefulSet update.

The important error was:

```text
UPGRADE FAILED:
cannot patch "bankapp-mysql" with kind StatefulSet
```

This was an important troubleshooting lesson:

> Helm does not bypass Kubernetes resource restrictions.

The failed upgrade was preserved in Helm's release history.

---

## 10. Helm History

Release history was inspected using:

```bash
helm history bankapp-mysql
```

The exercise produced revisions representing:

```text
1 → Install
2 → Upgrade
3 → Upgrade
4 → Failed upgrade
5 → Successful upgrade
6 → Rollback
```

This demonstrates that Helm maintains a history of release changes.

---

## 11. Helm Rollback

The release was rolled back to revision 3:

```bash
helm rollback bankapp-mysql 3
```

Helm created a new revision rather than changing the revision number back to 3.

Final state:

```text
Revision 6 → Rollback to 3
```

Verification:

```bash
helm list
kubectl get pods
```

The MySQL pod remained healthy:

```text
bankapp-mysql-0   1/1   Running
```

---

## 12. Inspect the Chart

Chart metadata:

```bash
helm show chart bitnami/mysql
```

Important information:

```text
Name:       mysql
Version:    14.0.3
AppVersion: 9.4.0
```

### Chart Version vs App Version

**Chart Version**

```text
14.0.3
```

This identifies the version of the Helm chart.

**App Version**

```text
9.4.0
```

This identifies the application version associated with the chart.

These two versions serve different purposes.

---

## 13. Inspect Default Values

```bash
helm show values bitnami/mysql
```

This displays the configuration options provided by the chart.

For example:

```yaml
metrics:
  enabled: false
```

and:

```yaml
global:
  security:
    allowInsecureImages: false
```

---

## 14. Render Templates Without Installing

Helm can render the Kubernetes manifests without creating resources:

```bash
helm template test-mysql bitnami/mysql \
  --set auth.database=bankappdb \
  --set global.security.allowInsecureImages=true \
  --set image.repository=bitnamilegacy/mysql \
  --set image.tag=9.4.0-debian-12-r1 \
  --set metrics.enabled=false
```

The rendered output included Kubernetes resources such as:

- NetworkPolicy
- PodDisruptionBudget
- ServiceAccount
- Secret
- ConfigMap
- StatefulSet
- Service

This is useful for understanding what a Helm chart generates before deployment.

---

## 15. Get Release Values

```bash
helm get values bankapp-mysql
```

This displays the values supplied to the installed release.

---


## Raw Kubernetes YAML vs Helm

Traditional Kubernetes deployments often require managing multiple YAML files individually:

```text
Deployment
Service
ConfigMap
Secret
PVC
PV
HPA
StatefulSet
Gateway
```

Helm adds a templating and release-management layer.

Instead of modifying multiple manifests manually, configuration can be managed through:

```text
values.yaml
```

and reused across different environments.

---

## Troubleshooting Lessons

### ImagePullBackOff

A Helm chart can install successfully while a pod still fails because a referenced container image cannot be pulled.

Always verify:

```bash
kubectl get pods
```

and investigate further with:

```bash
kubectl describe pod <pod-name>
```

### StatefulSet Restrictions

Some StatefulSet fields cannot be changed through a normal upgrade.

Helm upgrades are still subject to Kubernetes API rules.

### Helm Status vs Pod Status

A Helm release showing:

```text
STATUS: deployed
```

does not replace Kubernetes-level verification.

Always check:

```bash
kubectl get pods
kubectl get pvc
```

---


## Key Takeaways

- Helm packages Kubernetes applications into reusable charts.
- Releases provide lifecycle management for deployed charts.
- Values allow configuration without modifying templates.
- `helm history` provides release revision tracking.
- `helm rollback` can restore a previous release configuration.
- `helm template` helps inspect generated Kubernetes manifests.
- Kubernetes resource limitations still apply during Helm upgrades.
- Pod-level verification is essential after Helm operations.
- Credentials should never be committed to public repositories.

---

## Final Verification

```bash
helm list
```

```bash
helm history bankapp-mysql
```

```bash
kubectl get pods
```

Final expected state:

```text
bankapp-mysql   deployed
bankapp-mysql-0 1/1 Running
```

---

## Conclusion

This hands-on exercise provided practical experience with Helm beyond simply installing a chart.

I practiced the complete release lifecycle:

```text
Repository
    ↓
Chart
    ↓
Install
    ↓
Configure
    ↓
Upgrade
    ↓
Troubleshoot
    ↓
History
    ↓
Rollback
    ↓
Verify
```

The exercise also demonstrated how Helm and Kubernetes work together when real deployment issues occur.
