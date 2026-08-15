# Day 77 — Observability Project: Full Stack Implementation

## 📋 Overview

Day 77 marks the culmination of a five-day observability journey (Days 73–77). Over that stretch, a complete observability stack was built up piece by piece — metrics, then container/host visibility, then logs, then traces — and today all of it was assembled into one production-style architecture using a reference Docker Compose repository.

The final stack runs **8 services** working together:

| Service | Role |
|---|---|
| **Prometheus** | Metrics collection and storage (pull-based scraping) |
| **Node Exporter** | Host-level system metrics (CPU, memory, disk) |
| **cAdvisor** | Per-container resource metrics |
| **Grafana** | Unified visualization layer for metrics, logs, and traces |
| **Loki** | Log aggregation (label-based, not full-text indexed) |
| **Promtail** | Log collection agent that ships container logs to Loki |
| **OTEL Collector** | Receives, processes, and exports distributed traces (and metrics) |
| **Notes App** | Sample application generating real telemetry (a Django + React app) |

---

## 🧠 Theory: The Three Pillars of Observability

Observability is usually described through three complementary signal types. Each answers a different question about a running system:

### 1. Metrics
Metrics are numeric measurements sampled over time (e.g., CPU %, request rate, memory usage). They are cheap to store, easy to aggregate, and ideal for dashboards and alerting thresholds.

- **Prometheus** uses a **pull-based** model: it scrapes `/metrics` HTTP endpoints on a configured interval (here, every 15s) rather than having services push data to it.
- Each scrape target exposes metrics in a plain-text exposition format that Prometheus parses and stores as time series, each identified by a metric name plus a set of key/value **labels**.
- **PromQL** is the query language used to aggregate, rate, and threshold this data (e.g., `rate(...[5m])` to compute a per-second rate over a 5-minute window).

**Why it matters:** metrics tell you *that* something is wrong (CPU spiked, error rate rose) and are the backbone of alerting, but they don't tell you *why*.

### 2. Logs
Logs are discrete, timestamped event records — the most granular and highest-volume signal.

- **Loki** takes a different approach from tools like Elasticsearch: instead of full-text indexing every log line, it **indexes only labels** (e.g., `container_name`, `job`) and stores the log content itself as compressed chunks. This makes it dramatically cheaper to run at scale, at the cost of full-text search performance.
- **Promtail** is the agent that tails log sources (in this case, Docker container logs) and ships them to Loki with the right labels attached.
- **LogQL** is Loki's query language — a label selector (like PromQL's) combined with optional line filters (`|= "error"`) and metric extraction (`rate(...)`).

**Why it matters:** logs tell you the specific detail of *what happened* at the moment something went wrong.

### 3. Traces
Traces capture the path of a single request as it moves through a system, broken into **spans** — one span per unit of work (an HTTP handler, a database query, a downstream call).

- Spans carry a shared **traceId** (identifying the whole request) and each has its own **spanId**, with a **parentSpanId** linking it back to the span that triggered it. This is what lets a tracing UI reconstruct a waterfall/flame graph of the request.
- The **OTEL (OpenTelemetry) Collector** is vendor-neutral middleware: it *receives* spans (via the OTLP protocol, over HTTP on port 4318 or gRPC on 4317), *processes* them (batching, filtering, enriching), and *exports* them onward — to a debug console, a backend like Tempo/Jaeger, or a metrics pipeline.
- In this stack, the collector is configured with a `debug` exporter, which simply prints received spans to its own logs — useful for validation, but not for persistent trace storage (Tempo would be the production choice).

**Why it matters:** traces tell you *where in the call chain* time was spent or a failure occurred — especially critical in a microservices/multi-service architecture where a single user request touches several components.

### How the three pillars connect

```
Request hits Notes App
        │
        ├── generates metrics ──────► Prometheus ──┐
        ├── writes log lines ───────► Promtail ─► Loki ──┤
        └── emits spans ────────────► OTEL Collector ─────┼──► Grafana (single pane of glass)
                                                            │
Node Exporter (host) ──────────────► Prometheus ───────────┘
cAdvisor (containers) ─────────────► Prometheus ───────────┘
```

Grafana is the convergence point: one UI, three datasources (Prometheus for metrics, Loki for logs, and — in a production setup — Tempo for traces), letting an engineer pivot from "this metric spiked" → "here are the logs at that timestamp" → "here's the trace that shows exactly which downstream call was slow."

---

## 🚀 Task 1: Stack Deployment

### Launch

```bash
git clone https://github.com/LondheShubham153/observability-for-devops.git
cd observability-for-devops
docker compose up -d
```

### Verify

```bash
docker compose ps
```

### 📸 Screenshot 1: All 8 Services UP

<img width="1667" height="888" alt="all 8 services UP" src="https://github.com/user-attachments/assets/805d73c8-4828-4c91-bc62-3d91c57230a5" />


*All services healthy and running: Prometheus, Node Exporter, cAdvisor, Grafana, Loki, Promtail, OTEL Collector, and Notes App.*

### Service Endpoints

| Service | Port | Status |
|---|---|---|
| Prometheus | 9090 | ✅ UP |
| Node Exporter | 9100 | ✅ UP |
| cAdvisor | 8080 | ✅ UP |
| Grafana | 3000 | ✅ UP |
| Loki | 3100 | ✅ UP |
| Promtail | 9080 | ✅ UP |
| OTEL Collector | 4317 / 4318 | ✅ UP |
| Notes App | 8000 | ✅ UP |

---

## 📊 Task 2: Metrics Pipeline Validation

All 4 Prometheus scrape jobs report **UP** at `http://localhost:9090/targets`:

- `prometheus` — self-monitoring
- `node-exporter` — host metrics
- `docker` / `cadvisor` — container metrics
- `otel-collector` — OTLP metrics

### Key PromQL Queries

```promql
# All targets health
up

# CPU Usage
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory Usage
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100

# Container CPU
rate(container_cpu_usage_seconds_total{name!=""}[5m]) * 100

# Top 3 memory-hungry containers
topk(3, container_memory_usage_bytes{name!=""})
```

### Configuration Comparison

| Component | My Version (Days 73–74) | Reference Repo |
|---|---|---|
| Scrape Interval | 15s | 15s |
| Jobs | 4 scrape jobs | 4 scrape jobs |
| Alert Rules | Basic | Production-ready |
| Service Discovery | `static_configs` | `static_configs` |

---

## 📝 Task 3: Logs Pipeline Validation

### Generate Traffic

```bash
for i in $(seq 1 50); do
  curl -s http://localhost:8000 > /dev/null
  curl -s http://localhost:8000/api/ > /dev/null
done
```

### LogQL Queries Tested

```logql
# All logs
{job="docker"}

# Application logs only
{container_name="notes-app"}

# Error detection
{job="docker"} |= "error"

# HTTP request logs from the app
{container_name="notes-app"} |= "GET"

# Log rate by container
sum by (container_name) (rate({job="docker"}[5m]))
```

### Promtail vs Reference Comparison

| Feature | My Version (Day 75) | Reference Repo |
|---|---|---|
| Scrape Configs | `file_sd` | `file_sd` |
| Log Sources | `/var/log/*.log` | `/var/log/*.log` |
| Labeling | Basic | Enhanced with container labels |
| Pipeline Stages | Simple | Advanced transformation |

---

## 🎯 Task 4: Traces Pipeline Validation

### Send Test Trace

```bash
curl -X POST http://localhost:4318/v1/traces \
  -H "Content-Type: application/json" \
  -d '{
    "resourceSpans": [{
      "resource": {
        "attributes": [{
          "key": "service.name",
          "value": { "stringValue": "notes-app" }
        }]
      },
      "scopeSpans": [{
        "spans": [
          {
            "traceId": "aaaabbbbccccdddd1111222233334444",
            "spanId": "1111222233334444",
            "name": "GET /api/notes",
            "kind": 2,
            "startTimeUnixNano": "1700000000000000000",
            "endTimeUnixNano": "1700000000150000000",
            "attributes": [
              {"key": "http.method", "value": {"stringValue": "GET"}},
              {"key": "http.route", "value": {"stringValue": "/api/notes"}},
              {"key": "http.status_code", "value": {"intValue": "200"}}
            ]
          },
          {
            "traceId": "aaaabbbbccccdddd1111222233334444",
            "spanId": "5555666677778888",
            "parentSpanId": "1111222233334444",
            "name": "SELECT notes FROM database",
            "kind": 3,
            "startTimeUnixNano": "1700000000020000000",
            "endTimeUnixNano": "1700000000120000000",
            "attributes": [
              {"key": "db.system", "value": {"stringValue": "sqlite"}},
              {"key": "db.statement", "value": {"stringValue": "SELECT * FROM notes"}}
            ]
          }
        ]
      }]
    }]
  }'
```

This simulates a two-span trace: an HTTP request (`GET /api/notes`) that triggers a downstream database query (`SELECT notes FROM database`), linked via `parentSpanId`.

### Check the Debug Output

```bash
docker logs otel-collector 2>&1 | grep -A 20 "GET /api/notes"
```

### 📸 Screenshot 2: OTEL Collector Debug Output


<img width="1818" height="792" alt="image" src="https://github.com/user-attachments/assets/adbe5892-f51e-466a-b108-bdf793ddaf82" />


*Successful trace collection showing the parent-child span relationship, with HTTP request and database query spans and their attributes.*

### OTEL Collector Comparison

| Feature | My Version (Day 76) | Reference Repo |
|---|---|---|
| Receivers | `otlp` | `otlp` |
| Processors | `batch`, `memory_limiter` | `batch`, `memory_limiter`, `attributes` |
| Exporters | `debug` | `debug`, `prometheus` |
| Pipeline | traces only | traces + metrics |

---

## 🖥️ Task 5: Unified "Production Overview" Dashboard

### Dashboard Layout

**Row 1 — System Health (Node Exporter)**

| Panel | Type | Query |
|---|---|---|
| CPU Usage | Gauge | `100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)` |
| Memory Usage | Gauge | `(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100` |
| Disk Usage | Gauge | `(1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100` |
| Targets Up | Stat | `sum(up) / count(up)` |

**Row 2 — Container Metrics (cAdvisor)**

| Panel | Type | Query |
|---|---|---|
| Container CPU | Time series | `rate(container_cpu_usage_seconds_total{name!=""}[5m]) * 100` (legend `{{name}}`) |
| Container Memory | Bar chart | `container_memory_usage_bytes{name!=""} / 1024 / 1024` (legend `{{name}}`) |
| Container Count | Stat | `count(container_last_seen{name!=""})` |

**Row 3 — Application Logs (Loki)**

| Panel | Type | Query |
|---|---|---|
| App Logs | Logs | `{container_name="notes-app"}` |
| Error Rate | Time series | `sum(rate({job="docker"} \|= "error" [5m]))` |
| Log Volume | Time series | `sum by (container_name) (rate({job="docker"}[5m]))` |

**Row 4 — Service Overview**

| Panel | Type | Query |
|---|---|---|
| Prometheus Scrape Duration | Time series | `prometheus_target_interval_length_seconds{quantile="0.99"}` |
| OTEL Metrics Received | Stat | `otelcol_receiver_accepted_metric_points` |

### 📸 Screenshot 3: Production Overview Dashboard

<img width="1648" height="895" alt="dashboard" src="https://github.com/user-attachments/assets/2bb7df3e-5a39-4abd-a83b-baef16a8735f" />


*Complete unified dashboard showing system health, container metrics, application logs, and service overview.*

### Dashboard Configuration

- **Title:** "Production Overview — Observability Stack"
- **Time Range:** Last 30 minutes
- **Auto-Refresh:** Every 10 seconds

---

## 📊 Task 6: Complete Stack Comparison

### Full Configuration Comparison Matrix

| Component | My Implementation | Reference Repo | Key Differences |
|---|---|---|---|
| `prometheus.yml` | Days 73–74 | Root directory | Similar scrape configs; reference has richer alert rules |
| `loki-config.yml` | Day 75 | `loki/` directory | Basic storage vs. production-ready with retention |
| `promtail-config.yml` | Day 75 | `promtail/` directory | Simple scrapes vs. advanced pipeline stages |
| `otel-collector-config.yml` | Day 76 | `otel-collector/` directory | Basic vs. production processor chain |
| `datasources.yml` | Day 74 | `grafana/provisioning/` | Both auto-provisioned |
| `docker-compose.yml` | Days 73–76 | Root directory | Complete 8-service integration |

### Learning Journey Map

| Day | Topic | Implementation |
|---|---|---|
| 73 | Prometheus & PromQL | Setup Prometheus, scrape configs, basic queries |
| 74 | Node Exporter, cAdvisor, Grafana | Host metrics, container metrics, first dashboards |
| 75 | Loki & Promtail | Log aggregation, LogQL, log-metric correlation |
| 76 | OTEL Collector & Alerting | Trace collection, alert rules, span analysis |
| 77 | Full Stack Integration | All 8 services, unified dashboard, production architecture |

---

## 🏗️ Architecture Diagram

```
                         ┌─────────────────────────┐
                         │        Notes App          │
                         │   (Django + React, :8000) │
                         └───────────┬───────────────┘
                    metrics          │ logs              │ traces
             (scraped)               │ (stdout)           │ (OTLP push)
                    │                │                    │
                    ▼                ▼                    ▼
          ┌──────────────┐   ┌──────────────┐    ┌──────────────────┐
          │ Node Exporter │   │  Promtail    │    │  OTEL Collector   │
          │  cAdvisor     │   │ (log agent)  │    │ (:4317 / :4318)   │
          └──────┬────────┘   └──────┬───────┘    └─────────┬─────────┘
                 │ scrape             │ push                 │ debug export
                 ▼                    ▼                      ▼
          ┌──────────────┐   ┌──────────────┐        (console output;
          │  Prometheus   │   │     Loki      │         Tempo in prod)
          │   (:9090)     │   │   (:3100)     │
          └──────┬────────┘   └──────┬────────┘
                 │                    │
                 └─────────┬──────────┘
                            ▼
                    ┌──────────────┐
                    │   Grafana     │
                    │   (:3000)     │
                    │ Unified view: │
                    │ metrics+logs  │
                    └──────────────┘
```

---

## 🛠️ Production Readiness Additions

### What I Would Add for Production

**1. Alertmanager** — route Prometheus alerts to Slack/PagerDuty:

```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']
```

**2. Grafana Tempo for Trace Storage** — replace the `debug` exporter with Tempo for persistent, queryable trace storage instead of console-only output.

**3. Security Enhancements**
- HTTPS/TLS for all endpoints
- Basic auth for Grafana and Prometheus
- Network isolation between services

**4. Data Management**
- Log retention policies in Loki
- Prometheus retention and compaction settings
- Regular backup strategies

**5. High Availability**
- Multiple replicas of critical services
- Load balancing for Prometheus and Loki
- Data replication strategies

### Comparison with Managed Solutions

| Feature | Self-Managed | Datadog | AWS CloudWatch |
|---|---|---|---|
| Cost | Free (OSS) | Per-host | Pay-per-usage |
| Data Control | Full | Limited | AWS-managed |
| Customization | Unlimited | API-limited | AWS-specific |
| Setup Complexity | High | Low | Medium |
| Scalability | Manual | Auto | Auto |

---

## 📝 Key Takeaways

### Technical Insights
- **Metrics:** the pull-based Prometheus model works efficiently once targets are correctly configured and labeled.
- **Logs:** Loki's label-based indexing (rather than full-text indexing) is a better fit for high-volume containerized environments.
- **Traces:** the OTEL Collector provides a vendor-neutral, unified way to receive and route distributed tracing data.
- **Integration:** Docker Compose makes an otherwise complex multi-service observability stack manageable in a single command.

### Architecture Lessons
- **Data flow:** Metrics → Prometheus, Logs → Loki, Traces → OTEL Collector, all converging in Grafana as the single pane of glass.
- **Service discovery:** using container names as network endpoints (via the shared Docker network) simplifies configuration considerably.
- **Provisioning:** auto-provisioning datasources and dashboards (via `grafana/provisioning/`) removes manual setup steps and reduces the chance of misconfiguration.

### Best Practices Learned
- **Scrape intervals:** balance data freshness against resource/storage overhead (15s worked well here).
- **Label strategy:** consistent labeling across metrics, logs, and traces is what makes cross-signal correlation in Grafana actually work.
- **Retention:** proper retention settings prevent unbounded storage growth.
- **Alerting:** meaningful thresholds (not overly sensitive ones) are essential to avoid alert fatigue.

---

## 🧹 Clean Up

```bash
# Stop and remove all containers
docker compose down -v

# Verify cleanup
docker compose ps
```

> **Note:** the `-v` flag removes named volumes (Prometheus data, Grafana data, Loki data). Only use this once you're completely done exploring.

---

## 🔗 References

- [Observability for DevOps — Reference Repo](https://github.com/LondheShubham153/observability-for-devops)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Loki Documentation](https://grafana.com/docs/loki/)
- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)

---
