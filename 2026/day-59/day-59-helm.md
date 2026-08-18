# Day 59 – Helm — Kubernetes Package Manager ☸️

## What I learned today

Today I learned about **Helm**, which is a package manager for Kubernetes.

For the last few Kubernetes days, I was creating Deployments, Services, ConfigMaps, Secrets, PVCs, etc. using separate YAML files.

With Helm, these Kubernetes resources can be packaged into a **Chart** and managed as a single application.

The main concepts I learned today were:

* **Chart** – A package containing Kubernetes templates and configuration.
* **Release** – An installed instance of a Helm chart.
* **Repository** – A place where Helm charts are stored.

---

## 1. Installing and Checking Helm

I first checked that Helm was available in my environment.

```bash
helm version
helm env
```

`helm env` helped me understand some of the environment variables and configuration locations used by Helm.

---

## 2. Adding the Bitnami Repository

I added the Bitnami Helm repository:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Then updated the repository:

```bash
helm repo update
```

I also searched the repository:

```bash
helm search repo nginx
helm search repo bitnami
```

This helped me understand how Helm finds charts from repositories.

---

## 3. Deploying NGINX using Helm

I deployed NGINX using the Bitnami chart:

```bash
helm install my-nginx bitnami/nginx
```

Then I checked the resources created in Kubernetes:

```bash
kubectl get pods
kubectl get svc
```

My initial NGINX release created one running Pod.

The Service created for `my-nginx` was:

```text
Type: LoadBalancer
```

I also checked the Helm release:

```bash
helm list
helm status my-nginx
helm get manifest my-nginx
```

This helped me understand that Helm is generating and managing Kubernetes resources from chart templates.

---

## 4. Customizing a Helm Release

Next, I customized another NGINX release using `--set`.

```bash
helm install nginx-custom bitnami/nginx --set replicaCount=3 --set service.type=NodePort
```

I verified the result:

```bash
kubectl get pods
kubectl get svc
```

The result was:

```text
3 NGINX Pods
Service Type: NodePort
```

This was one of the important things I learned today.

Instead of changing multiple YAML files, I can override Helm values directly using:

```bash
--set key=value
```

For nested values:

```bash
--set service.type=NodePort
```

---

## 5. Using a Values File

I also created my own values file instead of passing everything through the command line.

My values included:

```yaml
replicaCount: 3

service:
  type: NodePort

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 250m
    memory: 256Mi
```

I installed the chart using:

```bash
helm install nginx-values bitnami/nginx -f custom-values.yml
```

I verified my custom values using:

```bash
helm get values nginx-values
```

Helm showed:

```text
replicaCount: 3
service:
  type: NodePort
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 250m
    memory: 256Mi
```

This helped me understand why values files are useful when we have multiple configuration options.

---

## 6. Upgrade and Rollback

Next, I upgraded my original `my-nginx` release.

I changed the replica count to 5:

```bash
helm upgrade my-nginx bitnami/nginx --set replicaCount=5
```

Helm created:

```text
Revision 2
```

I checked the release history:

```bash
helm history my-nginx
```

The history showed:

```text
Revision 1 → Install complete
Revision 2 → Upgrade complete
```

Then I tested rollback:

```bash
helm rollback my-nginx 1
```

After the rollback:

```bash
helm history my-nginx
```

showed:

```text
Revision 1 → superseded
Revision 2 → superseded
Revision 3 → Rollback to 1
```

This was an important concept for me:

**Rollback does not remove the previous revision.**

Instead, Helm creates a new revision.

```text
Revision 1
     ↓
Upgrade
     ↓
Revision 2
     ↓
Rollback to 1
     ↓
Revision 3
```

---

## 7. Creating My Own Helm Chart

After using an existing chart, I created my own Helm chart.

```bash
helm create my-app
```
The important files I focused on were:

### Chart.yaml

Contains information about the Helm chart.

### values.yaml

Contains default configuration values.

### templates/

Contains Kubernetes YAML templates that use Helm's templating syntax.

For example:

```yaml
replicas: {{ .Values.replicaCount }}
```

The value comes from:

```yaml
replicaCount: 3
```

---

## 8. Changing My Custom Chart

I changed my `values.yaml` so that the application used:

```yaml
replicaCount: 3
```

and:

```yaml
image:
  repository: nginx
  tag: "1.25"
```

Then I checked the chart using:

```bash
helm lint my-app
```

The result was:

```text
1 chart(s) linted, 0 chart(s) failed
```

There was only an informational message that an icon is recommended.

---

## 9. Previewing the Chart

Before installing the chart, I used:

```bash
helm template my-release ./my-app
```

This rendered the Kubernetes YAML without installing anything.

The generated Deployment showed:

```yaml
replicas: 3
```

and:

```yaml
image: "nginx:1.25"
```

I also saved the rendered output:

```bash
helm template my-release ./my-app > rendered.yaml
```

This helped me understand how Helm templates are converted into normal Kubernetes manifests.

---

## 10. Installing My Custom Chart

I installed my own chart:

```bash
helm install my-release ./my-app
```

Then checked:

```bash
helm list
kubectl get pods
kubectl get deployment
```

My Deployment showed:

```text
my-release-my-app   3/3
```

So the custom chart successfully created 3 replicas.

---

## 11. Upgrading My Custom Chart

I then upgraded my custom chart from 3 replicas to 5:

```bash
helm upgrade my-release ./my-app --set replicaCount=5
```

I checked the Deployment:

```bash
kubectl get deployment
```

The result was:

```text
my-release-my-app   5/5
```

I also checked:

```bash
helm history my-release
```

The release history showed:

```text
Revision 1 → Install complete
Revision 2 → Upgrade complete
```

---

## 12. Cleanup

After completing the practical, I cleaned up all the Helm releases:

```bash
helm uninstall my-nginx
helm uninstall nginx-custom
helm uninstall nginx-values
helm uninstall my-release
```

Finally:

```bash
helm list
```

showed no active releases.

I also checked:

```bash
kubectl get pods
```

and Kubernetes returned:

```text
No resources found in default namespace.
```

So the cleanup was successful.

---




