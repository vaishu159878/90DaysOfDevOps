# Day 75 -- Log Management with Loki, Promtail & Grafana

> **90DaysOfDevOps -- Observability**
>
> Centralized Docker container logging with **Promtail + Loki +
> Grafana**, with **Prometheus + cAdvisor** for metrics correlation.

## 📌 Project Overview

In this project, I built a practical observability stack on an AWS EC2
Ubuntu server.

The main goal was to collect Docker container logs, store them centrally
in Loki, query them using LogQL, visualize them in Grafana, and
correlate application logs with container CPU metrics from Prometheus.

### What I built

**Logging pipeline**

``` text
Docker Containers
       ↓
Docker JSON Logs
       ↓
   Promtail
       ↓
      Loki
       ↓
    Grafana
       ↓
     LogQL
```

**Metrics pipeline**

``` text
Docker Containers
       ↓
    cAdvisor
       ↓
   Prometheus
       ↓
    Grafana
       ↓
     PromQL
```

------------------------------------------------------------------------

## 🏗️ Architecture

``` text
                         AWS EC2 Ubuntu
                              |
                    +---------+---------+
                    |                   |
                    v                   v
             Docker Containers       cAdvisor
                    |                   |
                    |                   v
                    |              Prometheus
                    |                   |
                    v                   |
             Docker JSON Logs           |
                    |                   |
                    v                   |
                 Promtail               |
                    |                   |
                    v                   |
                   Loki <---------------+
                    |
                    v
                 Grafana
              /             \
          LogQL             PromQL
            ↓                  ↓
          Logs              Metrics
```

------------------------------------------------------------------------

## 🛠️ Technologies Used

  Technology   Purpose
  ------------ --------------------------------
  AWS EC2      Cloud server / lab environment
  Ubuntu       Server operating system
  Docker       Container runtime
  Promtail     Docker log collector
  Loki         Log aggregation and storage
  Grafana      Visualization and exploration
  Prometheus   Metrics collection
  cAdvisor     Docker container metrics
  LogQL        Loki query language
  PromQL       Prometheus query language

------------------------------------------------------------------------

## 🐳 Docker Compose Services

The final stack contains:

``` text
grafana
loki
notes-app
prometheus
promtail
cadvisor
```

Check the services with:

``` bash
docker compose ps
```

------------------------------------------------------------------------

## 📝 Promtail

Promtail collects Docker JSON logs and sends them to Loki.

The final configuration uses Docker service discovery:

``` yaml
scrape_configs:
  - job_name: docker

    docker_sd_configs:
      - host: unix:///var/run/docker.sock
        refresh_interval: 5s

    relabel_configs:
      - source_labels:
          - __meta_docker_container_name
        regex: '/(.*)'
        target_label: container_name

      - source_labels:
          - __meta_docker_container_log_stream
        target_label: stream

      - source_labels:
          - __meta_docker_container_id
        target_label: container_id

    pipeline_stages:
      - docker: {}
```

This allowed Loki to expose useful labels such as:

``` text
container_name
container_id
job
filename
service_name
stream
```

For example:

``` text
container_name=notes-app
```

### Verify Promtail

``` bash
curl http://localhost:9080/ready
```

Expected:

``` text
Ready
```

------------------------------------------------------------------------

## 🗄️ Loki

Loki runs on port `3100`.

Check Loki:

``` bash
curl http://localhost:3100/ready
```

Check available labels:

``` bash
curl -s http://localhost:3100/loki/api/v1/labels
```

Check container names:

``` bash
curl -s "http://localhost:3100/loki/api/v1/label/container_name/values"
```

------------------------------------------------------------------------

## 📊 Grafana

Grafana runs on port `3000`.

Open:

``` text
http://<EC2-PUBLIC-IP>:3000
```

The configured datasources are:

-   **Loki** → `http://loki:3100`
-   **Prometheus** → `http://prometheus:9090`

Grafana Explore was used to investigate application logs and metrics.

------------------------------------------------------------------------

## 🔎 LogQL Queries

### 1. All notes-app logs

``` logql
{container_name="notes-app"}
```

### 2. Search for 404 responses

``` logql
{container_name="notes-app"} |= "404"
```

### 3. Search for error messages

``` logql
{container_name="notes-app"} |= "error"
```

### 4. Count 404 errors per minute

``` logql
count_over_time({container_name="notes-app"} |= "404" [1m])
```

### 5. Calculate log rate

``` logql
rate({container_name="notes-app"}[5m])
```

### 6. Find HTTP 4xx/5xx responses

``` logql
{container_name="notes-app"} |~ "HTTP/1.1\" [45][0-9][0-9]"
```

------------------------------------------------------------------------

## 🧪 Generate Test Logs

Generate application traffic:

``` bash
for i in $(seq 1 50); do
    curl -s http://localhost:8000 > /dev/null
done
```

The notes application produced logs similar to:

``` text
"GET / HTTP/1.1" 200 651
```

The `/metrics` endpoint also produced HTTP 404 logs, which were useful
for LogQL filtering and error-counting exercises.

------------------------------------------------------------------------

## 📈 cAdvisor + Prometheus

Initially, Prometheus did not have:

``` text
container_cpu_usage_seconds_total
```

cAdvisor was added to expose Docker container metrics.

cAdvisor runs on port `8080`.

Test it with:

``` bash
curl -s http://localhost:8080/metrics | head
```

Prometheus then scrapes:

``` yaml
- job_name: cadvisor
  static_configs:
    - targets:
        - cadvisor:8080
```

------------------------------------------------------------------------

## 📊 PromQL

Check the container CPU metric:

``` promql
container_cpu_usage_seconds_total
```

For the notes application:

``` promql
rate(container_cpu_usage_seconds_total{name="notes-app"}[5m])
```

This produces a time-series graph showing the CPU usage rate of the
`notes-app` container.

------------------------------------------------------------------------

## 🔗 Logs + Metrics Correlation

One of the key exercises was correlating application logs with container
metrics.

### Loki

``` logql
rate({container_name="notes-app"}[5m])
```

This shows the rate of log entries from `notes-app`.

### Prometheus

``` promql
rate(container_cpu_usage_seconds_total{name="notes-app"}[5m])
```

This shows the CPU usage rate of the same container.

Together:

``` text
                 notes-app
                    |
          +---------+---------+
          |                   |
          v                   v
       Promtail            cAdvisor
          |                   |
          v                   v
         Loki             Prometheus
          |                   |
          +---------+---------+
                    |
                    v
                  Grafana
```

This provides both **what the application is logging** and **how the
container is behaving from a resource perspective**.

------------------------------------------------------------------------

## 🔧 Troubleshooting Highlights

### Problem 1: `container_name` label was missing

The initial Promtail setup used static file paths:

``` text
/var/lib/docker/containers/*/*-json.log
```

Logs were collected, but querying:

``` logql
{container_name="notes-app"}
```

returned no results.

### Solution

Switched Promtail to Docker service discovery and used relabeling:

``` yaml
docker_sd_configs:
  - host: unix:///var/run/docker.sock
```

After the change, `container_name` became available in Loki.

------------------------------------------------------------------------

### Problem 2: Prometheus had no container CPU metrics

The initial Prometheus query:

``` promql
container_cpu_usage_seconds_total
```

returned no data.

### Solution

Added cAdvisor and configured Prometheus to scrape:

``` text
cadvisor:8080
```

After that, the container CPU metric became available.

------------------------------------------------------------------------

### Problem 3: PromQL was initially entered in Loki

A PromQL query such as:

``` promql
rate(container_cpu_usage_seconds_total{name="notes-app"}[5m])
```

cannot be executed against the Loki datasource.

The solution was to select **Prometheus** as the Grafana datasource
before running PromQL.

------------------------------------------------------------------------

## ⚖️ Loki vs ELK

  -----------------------------------------------------------------------
  Feature                 Loki                    ELK
  ----------------------- ----------------------- -----------------------
  Log aggregation         Yes                     Yes

  Query language          LogQL                   KQL / Elasticsearch
                                                  Query DSL

  Visualization           Grafana                 Kibana

  Main indexing approach  Labels / metadata       Elasticsearch indexing

  Metrics integration     Strong Grafana          Requires additional
                          ecosystem integration   integrations

  Architecture            Loki + agent + Grafana  Elasticsearch +
                                                  Logstash/Beats + Kibana

  Typical use             Cloud-native            Centralized enterprise
                          observability stacks    log analytics
  -----------------------------------------------------------------------

