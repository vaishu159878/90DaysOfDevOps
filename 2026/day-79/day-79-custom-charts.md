# Day 79 — Creating a Custom Helm Chart for AI-BankApp

## Overview

This project converts the raw Kubernetes manifests of **AI-BankApp** into a reusable custom Helm chart.


The goal was to replace repeated hard-coded Kubernetes YAML with a parameterized Helm chart that can be configured through `values.yaml`.

---

# 1. Helm Chart Structure

The final chart structure is:

```text
helm-chart/
└── bankapp/
    ├── Chart.yaml
    ├── values.yaml
    ├── .helmignore
    └── templates/
        ├── _helpers.tpl
        ├── NOTES.txt
        ├── configmap.yaml
        ├── secrets.yaml
        ├── storage.yaml
        ├── bankapp-deployment.yaml
        ├── mysql-deployment.yaml
        ├── ollama-deployment.yaml
        ├── services.yaml
        └── hpa.yaml
```

---

# 2. Raw Kubernetes Manifests vs Helm Templates

## Example 1 — BankApp Deployment

### Raw Kubernetes manifest

A raw manifest contains fixed values directly in the YAML:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bankapp
spec:
  replicas: 4
  template:
    spec:
      containers:
        - name: bankapp
          image: trainwithshubham/ai-bankapp-eks:latest
          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
```

### Helm template

The Helm version replaces hard-coded configuration with values:

```yaml
spec:
  replicas: {{ .Values.bankapp.replicaCount }}

containers:
  - name: bankapp
    image: "{{ .Values.bankapp.image.repository }}:{{ .Values.bankapp.image.tag }}"
    imagePullPolicy: {{ .Values.bankapp.image.pullPolicy }}

    resources:
      {{- toYaml .Values.bankapp.resources | nindent 12 }}
```

### Difference

| Raw YAML | Helm |
|---|---|
| Replica count is hard-coded | Replica count comes from `values.yaml` |
| Image is hard-coded | Repository and tag are configurable |
| Resources are hard-coded | Resources can be changed without editing templates |
| Difficult to reuse | Reusable across environments |

---

# 3. Example 2 — MySQL Configuration

### Raw Kubernetes approach

```yaml
env:
  - name: MYSQL_ROOT_PASSWORD
    valueFrom:
      secretKeyRef:
        name: bankapp-secret
        key: MYSQL_ROOT_PASSWORD

  - name: MYSQL_DATABASE
    valueFrom:
      configMapKeyRef:
        name: bankapp-config
        key: MYSQL_DATABASE
```

### Helm approach

The resource names and configuration are generated from Helm:

```yaml
env:
  - name: MYSQL_ROOT_PASSWORD
    valueFrom:
      secretKeyRef:
        name: {{ include "bankapp.fullname" . }}-secret
        key: MYSQL_ROOT_PASSWORD

  - name: MYSQL_DATABASE
    valueFrom:
      configMapKeyRef:
        name: {{ include "bankapp.fullname" . }}-config
        key: MYSQL_DATABASE
```

This allows the same chart to be installed with different release names and namespaces.

---

# 4. Example 3 — Optional Ollama

A major benefit of Helm is conditional resources.

```yaml
{{- if .Values.ollama.enabled }}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "bankapp.fullname" . }}-ollama
...
{{- end }}
```

When:

```yaml
ollama:
  enabled: true
```

the Ollama resources are rendered.

When:

```yaml
ollama:
  enabled: false
```

the Ollama Deployment, Service, and PVC are not rendered.

This was validated using:

```bash
helm template my-bankapp . \
  --set bankapp.image.tag=abc1234 \
  --set bankapp.replicaCount=2 \
  --set ollama.enabled=false
```

The rendered output contained the BankApp image:

```text
trainwithshubham/ai-bankapp-eks:abc1234
```

and no Ollama Deployment, Service, or PVC.

---

# 5. Complete `values.yaml`

The current values used for the Kind validation environment are:

```yaml
bankapp:
  replicaCount: 4

  image:
    repository: trainwithshubham/ai-bankapp-eks
    tag: "latest"
    pullPolicy: Always

  resources:
    requests:
      memory: "256Mi"
      cpu: "250m"
    limits:
      memory: "512Mi"
      cpu: "500m"

  service:
    type: ClusterIP
    port: 8080

  autoscaling:
    enabled: true
    minReplicas: 1
    maxReplicas: 2
    targetCPUUtilization: 70

mysql:
  enabled: true

  image:
    repository: mysql
    tag: "8.0"

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

  image:
    repository: ollama/ollama
    tag: "latest"

  model: tinyllama

  resources:
    requests:
      memory: "1Gi"
      cpu: "300m"
    limits:
      memory: "2Gi"
      cpu: "700m"

  persistence:
    size: 10Gi
    storageClass: gp3

config:
  mysqlDatabase: bankappdb
  ollamaUrl: ""

secrets:
  mysqlRootPassword: Test@123
  mysqlUser: root
  mysqlPassword: Test@123

storageClass:
  create: true
  name: gp3
  provisioner: ebs.csi.aws.com

gateway:
  enabled: false
  hostname: ""
  tls:
    enabled: false
```

## Values explanation

### BankApp

```yaml
bankapp:
  replicaCount: 4
```

Controls the desired number of BankApp replicas.

The Kind validation environment uses an HPA range of 1–2 replicas because the cluster has limited CPU resources.

### Image

```yaml
image:
  repository: trainwithshubham/ai-bankapp-eks
  tag: "latest"
  pullPolicy: Always
```

Separates the image repository and tag so a different application version can be supplied without modifying the Deployment template.

For example:

```bash
--set bankapp.image.tag=abc1234
```

renders:

```text
trainwithshubham/ai-bankapp-eks:abc1234
```

### Resources

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

Defines Kubernetes CPU and memory requests and limits for the BankApp container.

### Service

```yaml
service:
  type: ClusterIP
  port: 8080
```

The application is exposed internally through a ClusterIP Service on port 8080.

### Autoscaling

```yaml
autoscaling:
  enabled: true
  minReplicas: 1
  maxReplicas: 2
  targetCPUUtilization: 70
```

Controls the Horizontal Pod Autoscaler.

The HPA targets 70% average CPU utilization.

### MySQL

```yaml
mysql:
  enabled: true
```

Controls whether MySQL is deployed.

The MySQL configuration includes:

- MySQL image and version
- CPU and memory resources
- 5Gi persistent storage
- StorageClass configuration

### Ollama

```yaml
ollama:
  enabled: true
```

Controls the optional Ollama AI service.

The configuration includes:

- Ollama image
- `tinyllama` model
- CPU and memory resources
- 10Gi persistent storage

### Application configuration

```yaml
config:
  mysqlDatabase: bankappdb
  ollamaUrl: ""
```

These values are used by the ConfigMap.

### Secrets

```yaml
secrets:
  mysqlRootPassword: Test@123
  mysqlUser: root
  mysqlPassword: Test@123
```

These values are converted to Kubernetes Secret data using Helm's `b64enc` function.

> For a real production deployment, passwords should not be committed directly to Git. Use a secret-management solution or external secret mechanism.

### StorageClass

```yaml
storageClass:
  create: true
  name: gp3
  provisioner: ebs.csi.aws.com
```

Allows the chart to create an AWS EBS `gp3` StorageClass when required.

For the Kind test environment, the existing `standard` StorageClass was used during installation:

```bash
--set storageClass.create=false \
--set mysql.persistence.storageClass=standard \
--set ollama.persistence.storageClass=standard
```

### Gateway

Gateway configuration is included for future extension but disabled by default.

---

# 6. Helm Go Template Syntax Cheat Sheet

## `.Values`

Reads configuration from `values.yaml`.

```yaml
replicas: {{ .Values.bankapp.replicaCount }}
```

Example:

```yaml
image: "{{ .Values.bankapp.image.repository }}:{{ .Values.bankapp.image.tag }}"
```

---

## `if`

Conditionally renders resources or configuration.

```yaml
{{- if .Values.ollama.enabled }}
```

The contents are rendered only when the condition is true.

Example:

```yaml
{{- if .Values.ollama.enabled }}
# Ollama resources
{{- end }}
```

---

## `range`

Iterates over a list or map.

Example:

```yaml
{{- range .Values.allowedOrigins }}
- {{ . }}
{{- end }}
```

Useful when a value contains multiple items.

---

## `with`

Changes the current context to a nested value.

Example:

```yaml
{{- with .Values.bankapp.resources }}
resources:
  {{- toYaml . | nindent 2 }}
{{- end }}
```

This makes nested configuration easier to reference.

---

## `include`

Calls a named helper template.

Example:

```yaml
name: {{ include "bankapp.fullname" . }}
```

This is commonly used for consistent resource naming and labels.

---

## `toYaml`

Converts a Helm value structure into YAML.

Example:

```yaml
resources:
  {{- toYaml .Values.bankapp.resources | nindent 12 }}
```

This is useful for rendering nested maps such as resource requests and limits.

---

## `nindent`

Adds indentation and a newline.

Example:

```yaml
{{- toYaml .Values.bankapp.resources | nindent 12 }}
```

This keeps generated YAML correctly indented.

---

## `b64enc`

Base64-encodes a value.

Example:

```yaml
MYSQL_PASSWORD: {{ .Values.secrets.mysqlPassword | b64enc | quote }}
```

This is used for Kubernetes Secret data.

---

# 7. Validation

## Helm Lint

The chart was validated with:

```bash
cd ~/AI-BankApp-DevOps/helm-chart/bankapp
helm lint .
```

Result:

```text
==> Linting .
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```

The icon message is informational; the chart passed lint successfully.

---

# 8. Render the Kubernetes Manifests

The chart was rendered using:

```bash
helm template my-bankapp .
```

The rendered output contains:

```text
Secret
ConfigMap
StorageClass
MySQL PVC
Ollama PVC
MySQL Service
Ollama Service
BankApp Service
BankApp Deployment
MySQL Deployment
Ollama Deployment
HorizontalPodAutoscaler
```

Important rendered examples include:

```yaml
image: "trainwithshubham/ai-bankapp-eks:latest"
```

and:

```yaml
readinessProbe:
  httpGet:
    path: /actuator/health
    port: 8080
```

The rendered HPA uses:

```yaml
minReplicas: 1
maxReplicas: 2
```

with:

```yaml
averageUtilization: 70
```

---

# 9. Helm Deployment on Kind

The chart was installed on the Kind cluster using:

```bash
helm install my-bankapp bankapp/ \
  -n bankapp --create-namespace \
  --set storageClass.create=false \
  --set mysql.persistence.storageClass=standard \
  --set ollama.persistence.storageClass=standard
```

The deployed resources were verified with:

```bash
helm list -n bankapp
```

and:

```bash
kubectl get pods -n bankapp
```

The final pod status was:

```text
my-bankapp-cf67664b9-65p24           1/1   Running
my-bankapp-cf67664b9-j8j4r           1/1   Running
my-bankapp-mysql-7d596bf88d-9zqvf    1/1   Running
my-bankapp-ollama-6d9474fbb4-fbd2r   1/1   Running
```

The application health endpoint returned:

```json
{
  "status": "UP",
  "groups": ["liveness", "readiness"]
}
```

The application root returned HTTP `302` and redirected to:

```text
/login
```

This confirmed that the Spring Boot application was running successfully.

---


# 10. Disabling Ollama

Helm allows optional components to be disabled without editing Kubernetes templates.

Run:

```bash
helm template my-bankapp . \
  --set bankapp.image.tag=abc1234 \
  --set bankapp.replicaCount=2 \
  --set ollama.enabled=false
```

The result showed:

```text
trainwithshubham/ai-bankapp-eks:abc1234
```

and removed the Ollama-specific resources:

```text
Ollama Deployment
Ollama Service
Ollama PVC
```

The rendered resources still included:

```text
BankApp
MySQL
BankApp Service
MySQL Service
MySQL PVC
ConfigMap
Secret
HPA
```

This demonstrates one of the main benefits of Helm: **environment-specific configuration can be changed using values instead of copying and editing multiple Kubernetes YAML files.**

---

# 11. Troubleshooting During Deployment

## Issue 1 — Helm lint failed from the wrong directory

Running:

```bash
helm lint .
```

from the home directory produced:

```text
Error: stat Chart.yaml: no such file or directory
```

The solution was to enter the chart directory:

```bash
cd ~/AI-BankApp-DevOps/helm-chart/bankapp
```

Then:

```bash
helm lint .
```

passed successfully.

---

## Issue 2 — Kind cluster CPU constraints

The first deployment had Pending pods because the single Kind node did not have enough available CPU for all requested workloads.

The solution was to reduce the Ollama resources for the local Kind environment:

```yaml
requests:
  memory: "1Gi"
  cpu: "300m"

limits:
  memory: "2Gi"
  cpu: "700m"
```

The HPA range was also adjusted for the small local cluster:

```yaml
minReplicas: 1
maxReplicas: 2
```

After the Helm upgrade, all application pods became healthy.

---

## Issue 3 — HPA metrics

The HPA resource was successfully created, but CPU metrics were initially unavailable because the Kind cluster did not have the Metrics API available.

This is separate from the Helm chart rendering and deployment itself.

The HPA template was still successfully rendered:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
```

with a 70% CPU target.

---

# 12. Key Learnings

Through this exercise I learned how to:

- Create a custom Helm chart from existing Kubernetes manifests
- Separate configuration from templates
- Use `values.yaml` for reusable configuration
- Use Helm conditional rendering
- Create reusable resource names with helpers
- Render nested YAML using `toYaml` and `nindent`
- Encode Kubernetes Secret values using `b64enc`
- Preserve init containers and lifecycle hooks in Helm
- Parameterize container images and replicas
- Configure PVCs through Helm values
- Create optional components such as Ollama
- Validate charts with `helm lint`
- Preview Kubernetes resources with `helm template`
- Deploy and troubleshoot a Helm release on Kind
- Handle resource constraints in a local Kubernetes cluster


---

# Conclusion

The original AI-BankApp Kubernetes manifests were converted into a reusable Helm chart.

Instead of maintaining separate hard-coded manifests for every environment, the application configuration can now be controlled through `values.yaml` and Helm command-line overrides.

The chart was successfully linted, rendered, deployed on Kind, and validated with running BankApp, MySQL, and Ollama workloads.

The optional Ollama component was also successfully disabled through:

```bash
--set ollama.enabled=false
```

This demonstrates how Helm can make Kubernetes deployments **reusable, configurable, and easier to maintain**.
