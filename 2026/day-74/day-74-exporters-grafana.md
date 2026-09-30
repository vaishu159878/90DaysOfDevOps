# DevOps Observability Stack

A hands-on observability project built on an AWS EC2 Ubuntu server using **Prometheus, Node Exporter, cAdvisor, Grafana, Docker, and Docker Compose**.

The goal is to collect host and container metrics, query them with Prometheus, visualize them in Grafana, and provision the Grafana datasource and dashboard through configuration files.

## Architecture

```text
AWS EC2 (Ubuntu)
      |
      +--> Node Exporter ----+
      |                      |
      +--> cAdvisor ---------+--> Prometheus --> Grafana
      |                      |
      +--> Notes App --------+
```

## Technologies

- AWS EC2
- Ubuntu
- Docker
- Docker Compose
- Prometheus
- Node Exporter
- cAdvisor
- Grafana
- PromQL

## Services

| Service | Purpose | Port |
|---|---|---:|
| Prometheus | Metrics collection and querying | 9090 |
| Grafana | Visualization and dashboards | 3000 |
| cAdvisor | Container resource metrics | 8080 |
| Node Exporter | Host metrics | 9100 |
| Notes App | Sample application | 8000 |

## Project Structure

```text
.
├── docker-compose.yml
├── prometheus.yml
└── grafana
    ├── dashboards
    │   └── devops-observability-overview.json
    └── provisioning
        ├── dashboards
        │   └── dashboards.yml
        └── datasources
            └── prometheus.yml
```

## Grafana Dashboard

Dashboard: **DevOps Observability Overview**

The dashboard contains:

- CPU Usage %
- Memory Usage %
- Container CPU Usage
- Container Memory Usage
- Disk Usage %

### PromQL Queries

CPU:

```promql
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

Memory:

```promql
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
```

Container CPU:

```promql
rate(container_cpu_usage_seconds_total{id=~"/system.slice/docker-.*\.scope"}[5m]) * 100
```

Container Memory:

```promql
container_memory_usage_bytes{id=~"/system.slice/docker-.*\.scope"} / 1024 / 1024
```

Disk:

```promql
(1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100
```

> cAdvisor in this environment exposed container IDs instead of the expected `name` label, so the container queries were adapted to the labels actually available.

## Grafana Provisioning

Prometheus is provisioned through:

```text
grafana/provisioning/datasources/prometheus.yml
```

```yaml
apiVersion: 1

datasources:
  - name: prometheus
    uid: cfztiujdm4lj4b
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true
```

The dashboard provider is configured through:

```text
grafana/provisioning/dashboards/dashboards.yml
```

```yaml
apiVersion: 1

providers:
  - name: 'default'
    orgId: 1
    folder: ''
    type: file
    disableDeletion: false
    editable: true
    options:
      path: /var/lib/grafana/dashboards
```

The dashboard JSON is mounted into Grafana at:

```text
/var/lib/grafana/dashboards
```

## Setup

Verify Docker:

```bash
docker --version
docker compose version
```

Validate Compose:

```bash
docker compose config
```

Pull images:

```bash
docker compose pull
```

Start the stack:

```bash
docker compose up -d
```

Check containers:

```bash
docker compose ps
```

## Access

Replace `<EC2_PUBLIC_IP>` with your EC2 public IP.

Prometheus:

```text
http://<EC2_PUBLIC_IP>:9090
```

Prometheus targets:

```text
http://<EC2_PUBLIC_IP>:9090/targets
```

Grafana:

```text
http://<EC2_PUBLIC_IP>:3000
```

cAdvisor:

```text
http://<EC2_PUBLIC_IP>:8080
```

Node Exporter:

```text
http://<EC2_PUBLIC_IP>:9100/metrics
```

## Verification

Prometheus health:

```bash
curl -s http://localhost:9090/-/healthy
```

Expected:

```text
Prometheus Server is Healthy.
```

Prometheus readiness:

```bash
curl -s http://localhost:9090/-/ready
```

Expected:

```text
Prometheus Server is Ready.
```

Check targets:

```bash
curl -s http://localhost:9090/api/v1/targets | python3 -c '
import sys,json
d=json.load(sys.stdin)
for x in d["data"]["activeTargets"]:
    print(x["labels"].get("job"), "->", x["health"], "->", x["scrapeUrl"])
'
```

Expected important targets:

```text
cadvisor -> up
node-exporter -> up
prometheus -> up
```

The Notes App may show `down` if its `/metrics` endpoint returns HTTP 404. The container itself can still be running normally.

## Useful Commands

Start:

```bash
docker compose up -d
```

Stop:

```bash
docker compose down
```

Restart Grafana:

```bash
docker compose up -d grafana
```

View status:

```bash
docker compose ps
```

View Grafana logs:

```bash
docker compose logs grafana --tail=100
```

View Prometheus logs:

```bash
docker compose logs prometheus --tail=100
```

View resource usage:

```bash
docker stats
```

## Troubleshooting

### Dashboard not visible

```bash
ls -lh ~/grafana/dashboards/
```

```bash
docker compose exec grafana ls -lh /var/lib/grafana/dashboards/
```

```bash
docker compose logs grafana --tail=200 | grep -Ei 'provision|dashboard|error'
```

### Prometheus target is down

Inspect targets:

```bash
curl -s http://localhost:9090/api/v1/targets | python3 -m json.tool
```

Check Node Exporter:

```bash
curl -s http://localhost:9100/metrics | head
```

Check cAdvisor:

```bash
curl -s http://localhost:8080/metrics | head
```

### cAdvisor query returns no data

Inspect the labels exposed by cAdvisor:

```bash
curl -s http://localhost:8080/metrics | grep 'container_cpu_usage_seconds_total{' | head
```

Use the labels exposed by the installed cAdvisor version when writing PromQL queries.


## Future Improvements

- Add Alertmanager.
- Configure CPU and memory alerts.
- Add application-level Prometheus metrics.
- Add HTTPS and stronger Grafana authentication.
- Add Nginx as a reverse proxy.
- Automate deployment with Terraform.
- Add CI/CD with GitHub Actions.
- Deploy the observability stack to Kubernetes.

#
