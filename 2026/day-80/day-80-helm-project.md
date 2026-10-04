# Day 80 - Helm Project

## What I worked on

In the previous steps, I worked with Kubernetes YAML files and then created a custom Helm chart for the AI-BankApp.

For this project, I wanted to make the chart more useful for different environments.

Instead of creating separate Kubernetes YAML files for Dev, Staging, and Production, I used the same Helm chart with different values files.

The structure is:

```text
helm-chart/bankapp/
├── Chart.yaml
├── values.yaml
├── values-dev.yaml
├── values-staging.yaml
├── values-prod.yaml
└── templates/
    ├── _helpers.tpl
    ├── configmap.yaml
    ├── deployment.yaml
    ├── hpa.yaml
    ├── mysql-deployment.yaml
    ├── ollama-deployment.yaml
    ├── pvc.yaml
    ├── secret.yaml
    ├── service.yaml
    ├── services.yaml
    ├── storageclass.yaml
    └── tests/
        └── test-connection.yaml
```

---

## 1. Environment Values

I created three values files:

- `values-dev.yaml`
- `values-staging.yaml`
- `values-prod.yaml`

The idea is simple:

**Same chart + different values = different environment**

### Development

For Dev, I kept the configuration lightweight because I was running the application on a Kind cluster.

```yaml
bankapp:
  replicaCount: 1

  image:
    repository: trainwithshubham/ai-bankapp-eks
    tag: "latest"
    pullPolicy: Always

  resources:
    requests:
      memory: "256Mi"
      cpu: "100m"
    limits:
      memory: "512Mi"
      cpu: "250m"

  autoscaling:
    enabled: false

mysql:
  enabled: true

  resources:
    requests:
      memory: "512Mi"
      cpu: "250m"
    limits:
      memory: "800Mi"
      cpu: "500m"

  persistence:
    size: 2Gi
    storageClass: standard

ollama:
  enabled: true
  model: tinyllama

  resources:
    requests:
      memory: "1Gi"
      cpu: "500m"
    limits:
      memory: "1.5Gi"
      cpu: "1000m"

  persistence:
    size: 5Gi
    storageClass: standard

storageClass:
  create: false

gateway:
  enabled: false
```

I had to increase the MySQL memory for my local environment because the MySQL pod was getting OOMKilled with the smaller memory limit.

---

### Staging

For Staging, I increased the replicas and enabled HPA.

```yaml
bankapp:
  replicaCount: 2

  image:
    repository: trainwithshubham/ai-bankapp-eks
    tag: "v1.2.0"
    pullPolicy: IfNotPresent

  resources:
    requests:
      memory: "256Mi"
      cpu: "250m"
    limits:
      memory: "512Mi"
      cpu: "500m"

  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 3
    targetCPUUtilization: 75

mysql:
  enabled: true

  resources:
    requests:
      memory: "256Mi"
      cpu: "250m"
    limits:
      memory: "512Mi"
      cpu: "500m"

  persistence:
    size: 5Gi
    storageClass: gp3

ollama:
  enabled: true
  model: tinyllama

  resources:
    requests:
      memory: "2Gi"
      cpu: "900m"
    limits:
      memory: "2.5Gi"
      cpu: "1500m"

  persistence:
    size: 10Gi
    storageClass: gp3

secrets:
  mysqlRootPassword: StagingPass@456
  mysqlUser: root
  mysqlPassword: StagingPass@456

storageClass:
  create: true

gateway:
  enabled: false
```

---

### Production

For Production, I used more replicas, larger storage, HPA, and enabled the Gateway.

```yaml
bankapp:
  replicaCount: 4

  image:
    repository: trainwithshubham/ai-bankapp-eks
    tag: "v1.2.0"
    pullPolicy: IfNotPresent

  resources:
    requests:
      memory: "256Mi"
      cpu: "250m"
    limits:
      memory: "512Mi"
      cpu: "500m"

  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 4
    targetCPUUtilization: 70

mysql:
  enabled: true

  resources:
    requests:
      memory: "512Mi"
      cpu: "500m"
    limits:
      memory: "1Gi"
      cpu: "1000m"

  persistence:
    size: 20Gi
    storageClass: gp3

ollama:
  enabled: true
  model: tinyllama

  resources:
    requests:
      memory: "2Gi"
      cpu: "900m"
    limits:
      memory: "2.5Gi"
      cpu: "1500m"

  persistence:
    size: 10Gi
    storageClass: gp3

secrets:
  mysqlRootPassword: ProdSecure@789
  mysqlUser: root
  mysqlPassword: ProdSecure@789

storageClass:
  create: true

gateway:
  enabled: true
```

---

## 2. Environment Comparison

| Setting | Dev | Staging | Prod |
|---|---|---|---|
| BankApp replicas | 1 | 2 | 4 |
| HPA | Disabled | 2-3 replicas | 2-4 replicas |
| CPU target | - | 75% | 70% |
| Image | latest | v1.2.0 | v1.2.0 |
| MySQL storage | 2Gi | 5Gi | 20Gi |
| MySQL storage class | standard | gp3 | gp3 |
| Ollama storage | 5Gi | 10Gi | 10Gi |
| Gateway | Disabled | Disabled | Enabled |

This made it easier for me to understand how the same application can have different requirements in different environments.

---

# 3. Helm Hook

The task also introduced Helm hooks for database readiness.

The hook I worked with was a Job that checks whether MySQL is accepting connections on port `3306`.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "bankapp.fullname" . }}-db-ready
  namespace: {{ .Release.Namespace }}
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "0"
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: wait-for-mysql
          image: busybox:1.36
          command:
            - /bin/sh
            - -c
            - |
              until nc -z {{ include "bankapp.fullname" . }}-mysql 3306; do
                echo "Waiting for MySQL..."
                sleep 2
              done
              echo "MySQL is ready!"
```

### What the annotations mean

**`pre-install,pre-upgrade`**

Runs the hook before a Helm install or upgrade.

**`hook-weight: "0"`**

Controls the order when there are multiple hooks.

**`before-hook-creation`**

Removes the previous hook Job before creating another one.

### What I learned from this

I initially tried using this hook with MySQL managed by the same Helm chart.

I found a problem: during a fresh installation, the hook can run before the MySQL Service created by the chart exists.

So the hook can end up waiting for something that the same Helm release has not created yet.

Because of this, I removed the active pre-install hook from my working chart.

The application already has init containers that wait for MySQL and Ollama, and I used a Helm test to verify that the BankApp is healthy after deployment.

For me, this was an important practical lesson:

> A Helm hook should not create a dependency cycle with resources that the same release still needs to create.

---

# 4. Helm Test

I added a Helm test under:

```text
templates/tests/test-connection.yaml
```

The test checks:

```text
/actuator/health
```

of the BankApp service.

The Development deployment passed the test successfully.

```text
NAME: bankapp-dev
NAMESPACE: dev
STATUS: deployed
REVISION: 6
TEST SUITE: bankapp-dev-bankapp-test-connection
Phase: Succeeded
```

This was useful because I was not only checking whether the pods were running. I was also checking whether the application itself was responding correctly.

---

# 5. Checking the Helm Deployment

I also checked the Helm releases using:

```bash
helm list -A
```

---

# 6. Helm Lint

Before packaging the chart, I checked it with:

```bash
helm lint helm-chart/bankapp
```

The result was:

```text
1 chart(s) linted, 0 chart(s) failed
```

There was only an informational message recommending an icon.

So the chart passed the lint check.

---

# 7. Rendering Different Environments

I also rendered the chart separately for all three environments.

```text
Dev       -> values-dev.yaml
Staging   -> values-staging.yaml
Prod      -> values-prod.yaml
```

This helped me verify that the values were being applied correctly without directly deploying all three environments.

For example, the Production rendering showed:

```text
replicas: 4
minReplicas: 2
maxReplicas: 4
averageUtilization: 70
storageClassName: gp3
```

---

# 8. Packaging the Chart

After validating the chart, I updated the chart version:

```yaml
version: 0.2.0
appVersion: "1.1.0"
```

Then I packaged it.

The result was:

```text
bankapp-0.2.0.tgz
```

I also checked the package metadata and confirmed:

```text
name: bankapp
version: 0.2.0
appVersion: 1.1.0
```

So the chart is now packaged and ready to be distributed.

---

# 9. How I Would Integrate Helm With GitOps

The way I understand the GitOps flow is:

```text
Developer pushes code
        |
        v
GitHub Actions
        |
        v
Build Docker image
        |
        v
Tag image
        |
        v
Update Helm values
        |
        v
Git repository
        |
        v
Argo CD detects change
        |
        v
Helm chart is rendered
        |
        v
Kubernetes / EKS
```

Instead of changing a raw Kubernetes deployment file, the CI pipeline can update:

```text
helm-chart/bankapp/values-prod.yaml
```

For example, the image tag can be updated to a Git commit SHA.

Argo CD can then use:

```text
helm-chart/bankapp
```

as the source and use:

```text
values-prod.yaml
```

for the Production environment.

One thing I learned is that Argo CD can work directly with Helm charts. The CI pipeline does not need to manually render every Kubernetes YAML file and commit the rendered output.

---

# 10. Helm vs Raw Manifests vs Kustomize

| Approach | When I would use it | AI-BankApp example |
|---|---|---|
| Raw manifests | Small/simple deployment | Original `k8s/` files |
| Helm | Multiple environments and many configurable values | Current `helm-chart/bankapp` |
| Kustomize | Small changes/overlays on existing YAML | Patch the existing `k8s/` manifests |

### Raw manifests

Raw YAML is easy to understand when starting with Kubernetes.

But when Dev, Staging, and Production need different replicas, resources, storage, images, and HPA settings, maintaining many copied YAML files can become difficult.

### Helm

Helm worked well for this project because I could keep the Kubernetes templates in one place and move environment differences into values files.

### Kustomize

Kustomize would also be a good option if I wanted to keep the original manifests and apply environment-specific overlays instead of using Helm templates.

For this AI-BankApp project, I chose Helm because the application has several configurable components and multiple environments.

---

# 11. Production Secrets

One thing I would change before calling this production-ready is secrets management.

Currently, the learning environment contains database credentials in the values files.

I would **not store real production passwords in Git**.

For an AWS/EKS environment, one option I would use is:

```text
AWS Secrets Manager
        |
        v
External Secrets Operator
        |
        v
Kubernetes Secret
        |
        v
AI-BankApp
```

Other options I learned about are:

- Sealed Secrets
- HashiCorp Vault

For an AWS-based project, I would prefer AWS Secrets Manager with External Secrets Operator.

I would also use separate secrets for each environment and apply least-privilege IAM permissions.

---

# 12. Production Improvements I Would Add

If I continued improving this project for production, I would add:

- External Secrets Operator
- AWS Secrets Manager
- ResourceQuota
- LimitRange
- NetworkPolicy
- PodDisruptionBudget
- SecurityContext
- Better image security scanning
- Helm diff before upgrades
- Automated Helm linting in CI
- Automated testing
- Argo CD deployment
- Monitoring with Prometheus and Grafana

For production deployments, I would also prefer:

```bash
helm upgrade --install
```

with:

```text
--wait
--timeout
--atomic
```

because I want the deployment to wait for resources and automatically roll back if the upgrade fails.

---

# 13. What I Learned

This project helped me understand Helm beyond just the commands.

The main idea I learned is:

```text
Kubernetes Templates
        +
Environment Values
        =
Reusable Deployment
```

Instead of copying the same Kubernetes YAML for every environment, I can reuse the same chart.

The biggest difference is:

```text
Dev       -> smaller resources
Staging   -> more replicas + HPA
Production -> higher capacity + Gateway
```

but the underlying Helm templates stay the same.

I also learned something important from the hook issue: just because Helm provides a hook does not mean it should be used everywhere. The dependency between resources needs to be considered carefully.

---

# 14. Final Result

At the end of this project I have:

- Created Dev, Staging, and Production values
- Created a reusable Helm chart
- Added a Helm test
- Validated the chart with `helm lint`
- Rendered all three environments
- Packaged the chart
- Created `bankapp-0.2.0.tgz`
- Updated chart version to `0.2.0`
- Updated application version to `1.1.0`
- Pushed the Helm chart to my fork
- Understood how Helm can fit into the GitOps workflow

The final structure is:

```text
AI-BankApp-DevOps
│
├── k8s/
│
├── helm-chart/
│   └── bankapp/
│       ├── Chart.yaml
│       ├── values.yaml
│       ├── values-dev.yaml
│       ├── values-staging.yaml
│       ├── values-prod.yaml
│       └── templates/
│
└── bankapp-0.2.0.tgz
```

## My takeaway

Before this project, I mainly looked at Kubernetes resources one YAML file at a time.

Now I understand why Helm is useful when an application has multiple environments.

**Same application → same chart → different values → different environments.**

That is the main concept I am taking forward from this project.
