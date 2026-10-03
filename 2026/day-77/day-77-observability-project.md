# End-to-End Observability Stack

A production-style observability project built on **AWS EC2** using **Docker Compose**, combining metrics, logs, telemetry, and visualization in one stack.

## Architecture

```text
                         AWS EC2 / Ubuntu
                                |
                         Docker Compose
                                |
        +-----------------------+------------------------+
        |                       |                        |
        v                       v                        v
  Node Exporter             cAdvisor                Notes App
  Host metrics          Container metrics           :8000
        |                       |                        |
        +-----------+-----------+                        |
                    |                                    |
                    v                                    |
               Prometheus <------------------------------+
                  :9090
                    |
                    v
                 Grafana
                  :3000
                    ^
                    |
             OpenTelemetry Collector
               :4317 / :4318
                    |
                    v
             Prometheus exporter
                  :8889

Docker logs
    |
    v
 Promtail
    |
    v
   Loki
  :3100
    |
    v
 Grafana
```

## Stack

| Component | Purpose | Port |
|---|---|---:|
| AWS EC2 | Hosts the complete stack | — |
| Node Exporter | Host CPU, memory and filesystem metrics | 9100 |
| cAdvisor | Docker/container resource metrics | 8080 |
| Prometheus | Metrics collection and storage | 9090 |
| Grafana | Dashboards and visualization | 3000 |
| Promtail | Collects Docker container logs | 9080 |
| Loki | Log aggregation | 3100 |
| OpenTelemetry Collector | Receives OTLP telemetry and exports metrics | 4317/4318 |
| Notes App | Sample application used for traffic and telemetry | 8000 |

## Features Implemented

- Host CPU, memory and disk monitoring
- Docker container CPU and memory monitoring
- Prometheus target health monitoring
- Centralized container/application logging with Loki
- Log analysis using LogQL
- OpenTelemetry OTLP pipeline
- Grafana auto-provisioned Prometheus and Loki data sources
- Unified `Production Overview — Observability Stack` dashboard
- Dashboard configured for the last 30 minutes with 10-second refresh
- Troubleshooting based on the metrics and labels actually exposed by the running environment

## Prerequisites

- AWS EC2 Ubuntu instance
- Docker
- Docker Compose v2
- Git
- Python 3

Verify:

```bash
docker --version
docker compose version
git --version
python3 --version
```

## Clone and Start

```bash
git clone https://github.com/LondheShubham153/observability-for-devops.git
cd observability-for-devops

docker compose up -d
docker compose ps
```

Expected services:

```text
prometheus
node-exporter
cadvisor
grafana
loki
promtail
otel-collector
notes-app
```

## Health Checks

### Prometheus

```bash
curl -s http://localhost:9090/-/healthy
curl -s http://localhost:9090/-/ready
```

### Loki

```bash
curl -s http://localhost:3100/ready
```

### Grafana

```bash
curl -s http://localhost:3000/api/health
```

### OTEL Collector

```bash
docker logs --tail=50 otel-collector
```

## Prometheus Targets

The working environment contains four Prometheus scrape targets:

- `prometheus`
- `node-exporter`
- `docker` / cAdvisor
- `otel-collector`

Check them:

```bash
curl -s http://localhost:9090/api/v1/targets | python3 -m json.tool
```

Or open:

```text
http://<EC2-PUBLIC-IP>:9090/targets
```

Useful query:

```promql
up
```

## PromQL Queries

### Host CPU

```promql
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

### Host Memory

```promql
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
```

### Disk Usage

```promql
100 * (
  1 -
  node_filesystem_avail_bytes{mountpoint="/",fstype!~"tmpfs|overlay"}
  /
  node_filesystem_size_bytes{mountpoint="/",fstype!~"tmpfs|overlay"}
)
```

### Target Count

```promql
count(up)
```

### Target Health

```promql
up
```

### Container CPU

The cAdvisor version used in this environment does not expose the `name` label expected by some reference queries. The working selector is based on Docker cgroup IDs:

```promql
rate(
  container_cpu_usage_seconds_total{id=~"/system.slice/docker-.*"}[5m]
) * 100
```

### Container Memory

```promql
container_memory_usage_bytes{id=~"/system.slice/docker-.*"} / 1024 / 1024
```

### Running Container Count

```promql
count(container_memory_usage_bytes{id=~"/system.slice/docker-.*"})
```

## Loki / LogQL

Generate application traffic:

```bash
for i in $(seq 1 50); do
  curl -s http://localhost:8000 > /dev/null
  curl -s http://localhost:8000/api/ > /dev/null
done
```

### All Docker Logs

```logql
{job="docker"}
```

### GET Requests

```logql
{job="docker"} |= "GET"
```

### Error Logs

```logql
{job="docker"} |= "error"
```

### Log Volume

```logql
sum(count_over_time({job="docker"}[5m]))
```

> Note: In this environment Loki exposes labels such as `job`, `filename`, `service_name`, and `stream`. The reference query using `container_name` was therefore adapted to the labels actually available.

## OpenTelemetry

The collector accepts OTLP over:

```text
gRPC  :4317
HTTP  :4318
```

Metrics are exported for Prometheus on:

```text
:8889
```

The metrics pipeline is:

```text
OTLP
  |
  v
OpenTelemetry Collector
  |
  v
Prometheus Exporter :8889
  |
  v
Prometheus :9090
  |
  v
Grafana
```

### OTEL Metric Validation

A test metric was sent through the OTLP HTTP endpoint and successfully reached Prometheus:

```text
demo_requests_total = 50
```

Query:

```bash
curl -s 'http://localhost:9090/api/v1/query?query=demo_requests_total' \
| python3 -m json.tool
```

Clean output:

```bash
curl -s 'http://localhost:9090/api/v1/query?query=demo_requests_total' \
| python3 -c '
import sys,json
d=json.load(sys.stdin)
for r in d["data"]["result"]:
    print("Metric       :", r["metric"]["__name__"])
    print("Service      :", r["metric"].get("exported_job"))
    print("OTEL Scope   :", r["metric"].get("otel_scope_name"))
    print("Value        :", r["value"][1])
'
```

Expected:

```text
Metric       : demo_requests_total
Service      : notes-app
OTEL Scope   : observability-demo
Value        : 50
```

## Grafana Dashboard

Dashboard:

```text
Production Overview — Observability Stack
```

Configuration:

- Time range: Last 30 minutes
- Auto refresh: 10 seconds
- Prometheus datasource: provisioned automatically
- Loki datasource: provisioned automatically

### Dashboard Panels

- CPU Usage
- Memory Usage
- Disk Usage
- Prometheus Targets
- Container CPU Usage
- Container Memory Usage
- Running Containers
- Loki Log Volume
- Loki Error Rate
- Application Logs
- Prometheus Scrape Duration
- OTEL Metrics Received
- Prometheus Target Health

## Grafana Data Sources

The following data sources are provisioned automatically:

```yaml
Prometheus:
  url: http://prometheus:9090

Loki:
  url: http://loki:3100
```

Open Grafana:

```text
http://<EC2-PUBLIC-IP>:3000
```

## Troubleshooting Lessons

### cAdvisor `name` label missing

Reference queries using:

```promql
{name!=""}
```

returned no data in this environment.

The available Docker cgroup series were identified using:

```promql
{id=~"/system.slice/docker-.*"}
```

The dashboard was adapted to use the available labels instead of assuming the reference environment was identical.

### Loki `container_name` label missing

The environment exposed:

```text
filename
job
service_name
stream
```

Therefore the working queries use:

```logql
{job="docker"}
```

and filters such as:

```logql
{job="docker"} |= "GET"
```

### Prometheus scrape metrics

The reference dashboard query:

```promql
prometheus_target_scrape_duration_seconds
```

was not present in the running Prometheus environment.

The available metric was:

```promql
scrape_duration_seconds
```

This was used for the dashboard's Prometheus Scrape Duration panel.

### OTEL Collector shell access

The collector image does not contain `/bin/sh`, so commands such as:

```bash
docker exec otel-collector sh
```

do not work.

The collector was instead validated through its exposed OTLP endpoint, Prometheus scrape target, Prometheus query API, and container logs.


## Production Improvements

For a production deployment, I would consider:

- Alertmanager for alert routing to Slack/PagerDuty
- Grafana Tempo for trace storage
- HTTPS/TLS for exposed endpoints
- Authentication and network restrictions for Grafana and Prometheus
- Log retention and storage limits
- High availability with multiple Prometheus/Loki replicas
- Persistent and scalable object storage
- Centralized secrets management
- Resource limits for all containers
- Backup and disaster-recovery procedures
- AWS security groups restricted to required ports only



## Key Takeaways

This project brought together the observability concepts developed across the previous observability work:

| Stage | Focus |
|---|---|
| Metrics | Prometheus and PromQL |
| Infrastructure | Node Exporter and cAdvisor |
| Visualization | Grafana |
| Logging | Loki, Promtail and LogQL |
| Telemetry | OpenTelemetry Collector |
| Integration | Full stack with Docker Compose |
| Final outcome | Unified Production Overview dashboard |

The main practical lesson was that real environments do not always expose exactly the labels or metric names shown in a reference tutorial. Troubleshooting the actual telemetry, adapting queries, and validating each data flow end-to-end are essential Cloud/DevOps skills.



