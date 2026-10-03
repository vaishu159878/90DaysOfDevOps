# OpenTelemetry & Alerting

## Overview

This project demonstrates an end-to-end observability pipeline built on an AWS EC2 Ubuntu server.

The setup collects telemetry with **OpenTelemetry Collector**, exposes metrics for **Prometheus**, evaluates infrastructure alert rules, and connects **Grafana** to Prometheus for visualization and alerting.

### Architecture

```text
OTLP Application
      │
      ├── Metrics
      └── Traces
           │
           ▼
┌─────────────────────────┐
│ OpenTelemetry Collector │
│  OTLP gRPC :4317        │
│  OTLP HTTP :4318        │
│  Prometheus :8889       │
└────────────┬────────────┘
             │
             ▼
       ┌────────────┐
       │ Prometheus │
       │   :9090    │
       └─────┬──────┘
             │
       ┌─────┴──────────┐
       │                │
       ▼                ▼
 Alert Rules         Grafana
                    :3000
                       │
                       ▼
                Contact Point
                   Webhook
```

## Technologies Used

- AWS EC2
- Ubuntu
- Docker
- Docker Compose
- OpenTelemetry Collector
- Prometheus
- Grafana
- PromQL
- OTLP
- Webhook notification

## OpenTelemetry Collector

The Collector receives OTLP telemetry through:

- gRPC: `4317`
- HTTP: `4318`

The Prometheus exporter exposes metrics on:

```text
8889
```

Traces and logs are sent to the debug exporter for validation.

### Collector pipelines

```text
Metrics:
OTLP → Batch → Prometheus

Traces:
OTLP → Batch → Debug

Logs:
OTLP → Batch → Debug
```

## Prometheus

Prometheus scrapes:

```text
http://otel-collector:8889/metrics
http://localhost:9090/metrics
```

The scrape interval and rule evaluation interval are both configured for 15 seconds.

### Verify Prometheus

```bash
curl -s http://localhost:9090/-/healthy
```

Check targets:

```text
http://<EC2-PUBLIC-IP>:9090/targets
```

Expected targets:

```text
otel-collector    UP
prometheus        UP
```

## OTLP Trace Test

A test trace was sent to:

```text
POST http://localhost:4318/v1/traces
```

The Collector logs confirmed receipt of the test span.

Example verification:

```bash
docker logs otel-collector 2>&1 | grep -A 20 "test-span"
```

## OTLP Metric Test

A test metric named:

```text
test_requests_total
```

was sent to the Collector through OTLP HTTP.

Verification:

```bash
curl http://localhost:8889/metrics | grep test_requests
```

Expected metric:

```text
test_requests_total ... 42
```

Prometheus query:

```bash
curl -s 'http://localhost:9090/api/v1/query?query=test_requests_total'
```

The metric was successfully observed in Prometheus with a value of `42` during testing.

> **Note:** This was a test metric generated through a one-time OTLP request. It was not a continuously generated application metric, so it could disappear from Prometheus after the Collector no longer received the metric.

## Prometheus Alert Rules

The following alert rules were configured:

| Alert | Condition | Severity |
|---|---|---|
| HighCPUUsage | CPU usage above 80% for 2 minutes | warning |
| HighMemoryUsage | Memory usage above 85% for 2 minutes | warning |
| ContainerDown | `notes-app` container metric absent for 1 minute | critical |
| TargetDown | Prometheus scrape target is down for 1 minute | critical |
| HighDiskUsage | Root filesystem usage above 90% for 5 minutes | critical |

Rules are loaded through:

```yaml
rule_files:
  - /etc/prometheus/alert-rules.yml
```

Verify loaded rules:

```bash
curl -s http://localhost:9090/api/v1/rules
```

## Grafana

Grafana runs on:

```text
3000
```

Health check:

```bash
curl -s http://localhost:3000/api/health
```

Prometheus was configured as the Grafana data source:

```text
http://prometheus:9090
```

The Grafana data source health check returned:

```text
Successfully queried the Prometheus API.
status: OK
```

## Grafana Alerting

A Grafana alert rule named:

```text
Day76 Test Metric Alert
```

was created using:

```text
test_requests_total > 40
```

The rule uses a 1-minute evaluation period.

A webhook contact point was also configured:

```text
Day76-Webhook
```

The Grafana notification policy routes alerts to this contact point.

### Important testing note

During testing, Grafana correctly reported `DatasourceNoData` when the one-time test metric was no longer present in Prometheus.

The Prometheus data source itself remained healthy.

This demonstrated an important troubleshooting scenario:

```text
Grafana
   ↓
Prometheus data source
   ↓
Metric unavailable
   ↓
DatasourceNoData
```

## Troubleshooting Lessons

### Prometheus target verification

If a target is unavailable:

```bash
curl -s http://localhost:9090/api/v1/targets
```

or open:

```text
http://<EC2-PUBLIC-IP>:9090/targets
```

### Check OTEL Collector logs

```bash
docker logs otel-collector --tail 50
```

### Check Prometheus logs

```bash
docker logs prometheus --tail 50
```

### Check Grafana logs

```bash
docker logs grafana --tail 50
```

### Validate Docker Compose

```bash
docker compose config
```

### Check running services

```bash
docker ps
```


## Key Learnings

### 1. Telemetry collection

OpenTelemetry provides a standard way to receive and process metrics, traces, and logs.

### 2. Metrics pipeline

The metric flow is:

```text
OTLP → OpenTelemetry Collector → Prometheus → Grafana
```

### 3. Alerting

Monitoring is not only about dashboards. Alert rules turn telemetry into actionable signals.

### 4. Troubleshooting

When Grafana reported `DatasourceNoData`, the issue was traced backward through the monitoring pipeline and the Prometheus data source was verified independently.

### 5. Infrastructure as code mindset

The stack is defined using Docker Compose and configuration files, making the observability environment reproducible.

## Useful Commands

```bash
docker compose config
docker compose up -d
docker compose ps
docker ps

docker logs otel-collector --tail 50
docker logs prometheus --tail 50
docker logs grafana --tail 50

curl -s http://localhost:9090/-/healthy
curl -s http://localhost:3000/api/health
curl -s http://localhost:9090/api/v1/rules
```

