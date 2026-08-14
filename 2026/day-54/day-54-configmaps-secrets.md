# Day 54 -- Kubernetes ConfigMaps and Secrets

------------------------------------------------------------------------

## 1. ConfigMap from Literals

I created a ConfigMap called `app-config` with three configuration
values:

``` bash
kubectl create configmap app-config --from-literal=APP_ENV=production --from-literal=APP_DEBUG=false --from-literal=APP_PORT=8080
```

I verified it using:

``` bash
kubectl get configmap
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml
```

The ConfigMap contained:

``` text
APP_ENV=production
APP_DEBUG=false
APP_PORT=8080
```

One important thing I noticed is that ConfigMap values are stored as
plain text.

They are meant for **non-sensitive configuration**, not passwords or
other secrets.

------------------------------------------------------------------------

## 2. ConfigMap from a File

For the next task, I created a custom Nginx configuration file called
`default.conf`.

The configuration added a `/health` endpoint:

``` nginx
server {
    listen 80;
    server_name localhost;

    location / {
        return 200 "Hello from Nginx\n";
    }

    location /health {
        return 200 "healthy\n";
    }
}
```

Then I created a ConfigMap from the file:

``` bash
kubectl create configmap nginx-config \
  --from-file=default.conf=default.conf
```

I verified the contents using:

``` bash
kubectl get configmap nginx-config -o yaml
```

The key `default.conf` became the filename when the ConfigMap was
mounted inside the Pod.

------------------------------------------------------------------------

## 3. Using ConfigMap as Environment Variables

I created a Pod that used `envFrom` and `configMapRef`.

The important part of the manifest was:

``` yaml
envFrom:
  - configMapRef:
      name: app-config
```

After creating the Pod:

``` bash
kubectl apply -f configmap-env-pod.yaml
```

I checked the logs:

``` bash
kubectl logs configmap-env-pod
```

The output was:

``` text
APP_ENV=production
APP_DEBUG=false
APP_PORT=8080
```

This helped me understand that Kubernetes can inject all the keys from a
ConfigMap as environment variables.

------------------------------------------------------------------------

## 4. Mounting ConfigMap as a Volume

Next, I created an Nginx Pod and mounted `nginx-config` into:

``` text
/etc/nginx/conf.d
```

I verified the mounted file using:

``` bash
kubectl exec nginx-config-pod -- cat /etc/nginx/conf.d/default.conf
```

The custom Nginx configuration was present inside the container.

I also tested the health endpoint:

``` bash
kubectl exec nginx-config-pod -- curl -s http://localhost/health
```

Output:

``` text
healthy
```

This confirmed that the ConfigMap was successfully mounted and used by
Nginx.

### What I learned

For simple values, environment variables are convenient.

For complete configuration files, volume mounts are more useful.

------------------------------------------------------------------------

## 5. Creating a Kubernetes Secret

I created a Secret called `db-credentials`:

``` bash
kubectl create secret generic db-credentials \
  --from-literal=DB_USER=admin \
  --from-literal='DB_PASSWORD=s3cureP@ssw0rd'
```

I checked it with:

``` bash
kubectl get secrets
kubectl get secret db-credentials -o yaml
```

The values appeared as base64-encoded strings.

For example:

``` text
DB_USER: YWRtaW4=
```

I decoded the values using:

``` bash
kubectl get secret db-credentials \
  -o jsonpath='{.data.DB_USER}' | base64 --decode
```

and:

``` bash
kubectl get secret db-credentials \
  -o jsonpath='{.data.DB_PASSWORD}' | base64 --decode
```

The decoded values were:

``` text
admin
s3cureP@ssw0rd
```

### Important lesson

**Base64 is encoding, not encryption.**

A Kubernetes Secret should not be considered secure simply because its
values are displayed in base64.

Access control through RBAC and encryption at rest are important parts
of securing Secrets.

------------------------------------------------------------------------

## 6. Using Secret in a Pod

I created a Pod that used the Secret in two different ways.

### Environment variable

I injected `DB_USER` using `secretKeyRef`:

``` yaml
env:
  - name: DB_USER
    valueFrom:
      secretKeyRef:
        name: db-credentials
        key: DB_USER
```

### Volume mount

I mounted the complete Secret:

``` yaml
volumeMounts:
  - name: db-credentials-volume
    mountPath: /etc/db-credentials
    readOnly: true
```

The Secret was defined as:

``` yaml
volumes:
  - name: db-credentials-volume
    secret:
      secretName: db-credentials
```

After creating the Pod, I checked:

``` bash
kubectl logs secret-pod
```

I could see the Secret values through the environment variable and the
mounted files.

I also observed that the mounted Secret files contain the decoded
plaintext values, not the base64 strings shown in the Secret YAML.

------------------------------------------------------------------------

## 7. ConfigMap Live Update

This was the most interesting part of today's practice.

First, I created:

``` bash
kubectl create configmap live-config \
  --from-literal=message=hello
```

Then I created a Pod that continuously read the mounted file:

``` text
/etc/live-config/message
```

The Pod initially printed:

``` text
hello
hello
hello
```

Then I updated the ConfigMap:

``` bash
kubectl patch configmap live-config \
  --type merge \
  -p '{"data":{"message":"world"}}'
```

I kept the Pod running and watched its logs.

After some time, the output changed from:

``` text
hello
```

to:

``` text
world
```

My actual output showed:

``` text
14:10:59 UTC 2026: hello
14:11:04 UTC 2026: world
14:11:09 UTC 2026: world
```

The important part is that I **did not restart the Pod**.

The volume-mounted ConfigMap was updated automatically.

------------------------------------------------------------------------

## 8. Environment Variables vs Volume Mounts

One important difference I learned today:

  Method                 ConfigMap/Secret update automatically?
  ---------------------- -----------------------------------------------
  Environment variable   No
  Volume mount           Yes, for supported mounted ConfigMaps/Secrets

### Environment variable

``` text
ConfigMap
   ↓
Environment variable
   ↓
Container starts
```

The value is set when the container starts. Updating the ConfigMap does
not change the already-existing environment variable.

### Volume mount

``` text
ConfigMap
   ↓
Volume
   ↓
File inside Pod
```

When the ConfigMap changes, Kubernetes can update the mounted file
automatically.

------------------------------------------------------------------------

## 9. ConfigMap vs Secret

  -----------------------------------------------------------------------
  ConfigMap                           Secret
  ----------------------------------- -----------------------------------
  Used for non-sensitive              Used for sensitive configuration
  configuration                       

  Stored as plain text data           Values are represented as base64 in
                                      YAML

  Example: app environment            Example: database password

  Can be injected as env variables    Can be injected as env variables

  Can be mounted as files             Can be mounted as files
  -----------------------------------------------------------------------


## 10. Cleanup

After completing the practical work, I removed the resources I created:

``` bash
kubectl delete pod configmap-env-pod nginx-config-pod secret-pod live-config-pod

kubectl delete configmap app-config nginx-config live-config

kubectl delete secret db-credentials
```

I verified the Pods were removed:

``` bash
kubectl get pods
```

------------------------------------------------------------------------
