# Day 76 — OpenTelemetry and Alerting

## Introduction

Today I added the third pillar of observability — **traces** — using OpenTelemetry, and set up **alerting** so the system notifies me when something goes wrong instead of relying on manually watching dashboards.

By the end of this task, the observability stack covers all three pillars (metrics, logs, traces) and actively alerts on problems via Prometheus rules and Grafana notifications.

---

## Task 1 — Understand OpenTelemetry (Theory)

### What is OpenTelemetry (OTEL)?
A vendor-neutral, open-source framework for generating, collecting, and exporting telemetry data — metrics, logs, and traces. OTEL itself is **not a backend**; it collects data and ships it to backends like Prometheus, Jaeger, Loki, or Datadog.

### What is the OTEL Collector?
A standalone service that receives, processes, and exports telemetry. It has a three-stage pipeline:

- **Receivers** — accept incoming data (OTLP, Prometheus, Jaeger formats)
- **Processors** — transform data (batching, filtering, sampling)
- **Exporters** — send data onward to backends (Prometheus, debug console, Jaeger)

### What is OTLP?
**OpenTelemetry Protocol** — the standard wire format for sending telemetry data. It supports:
- **gRPC** on port `4317`
- **HTTP** on port `4318`

### What are distributed traces?
A trace follows a single request as it travels through multiple services. Each step is called a **span**.

Spans contain:
- Trace ID
- Span ID
- Parent span ID
- Start time
- Duration
- Attributes

**Example:**
```
User request → API Gateway (span 1) → Auth Service (span 2) → Database (span 3)
```

### Metrics vs Logs vs Traces

| Pillar | What it tells you | Example |
|---|---|---|
| Metrics | Aggregated numeric measurements over time | CPU usage, request count |
| Logs | Discrete, timestamped event records | Error message, request log line |
| Traces | The path and timing of a single request across services | Request latency breakdown per service |

---

## Task 2 — Add the OpenTelemetry Collector

### Collector Configuration
`otel-collector/otel-collector-config.yml`:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:

exporters:
  prometheus:
    endpoint: "0.0.0.0:8889"
  debug:
    verbosity: detailed

service:
  pipelines:
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [prometheus]
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [debug]
    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [debug]
```

**Explanation:**
- **Receivers** — accept OTLP data over gRPC (4317) and HTTP (4318)
- **Processors** — batches data before exporting to reduce overhead
- **Exporters**:
  - Metrics → Prometheus-compatible endpoint on port `8889` (scraped by Prometheus)
  - Traces & Logs → debug console output (in production these would go to Jaeger or Tempo)

### Docker Compose Addition
```yaml
  otel-collector:
    image: otel/opentelemetry-collector-contrib:latest
    container_name: otel-collector
    ports:
      - "4317:4317"   # OTLP gRPC
      - "4318:4318"   # OTLP HTTP
      - "8889:8889"   # Prometheus exporter
    volumes:
      - ./otel-collector/otel-collector-config.yml:/etc/otelcol-contrib/config.yaml
    restart: unless-stopped
```

### Prometheus Scrape Config
```yaml
  - job_name: "otel-collector"
    static_configs:
      - targets: ["otel-collector:8889"]
```

### Verification
```bash
docker compose up -d
docker logs otel-collector 2>&1 | tail -5
```
Confirmed `otel-collector` shows as **UP** under Prometheus → Status → Targets.

**Screenshot:** `otel-up.png` — OpenTelemetry Collector running successfully, exposed on OTLP (4317/4318) and Prometheus exporter (8889).


<img width="1680" height="845" alt="Otel  up" src="https://github.com/user-attachments/assets/b30189cf-94cb-4c99-ab11-2087a29cddef" />


---

## Task 3 — Send Test Traces and Metrics to the Collector

### Sending a Test Trace
```bash
curl -X POST http://localhost:4318/v1/traces \
  -H "Content-Type: application/json" \
  -d '{
    "resourceSpans": [{
      "resource": {
        "attributes": [{ "key": "service.name", "value": { "stringValue": "my-test-service" } }]
      },
      "scopeSpans": [{
        "spans": [{
          "traceId": "5b8efff798038103d269b633813fc60c",
          "spanId": "eee19b7ec3c1b174",
          "name": "test-span",
          "kind": 1,
          "startTimeUnixNano": "1544712660000000000",
          "endTimeUnixNano": "1544712661000000000",
          "attributes": [
            { "key": "http.method", "value": { "stringValue": "GET" } },
            { "key": "http.status_code", "value": { "intValue": "200" } }
          ]
        }]
      }]
    }]
  }'
```

Verify in collector logs:
```bash
docker logs otel-collector 2>&1 | grep -A 10 "test-span"
```

**Screenshot:** `otel-log-collector.png` — Collector debug logs showing the received OTLP trace (`test-span`).

<img width="891" height="543" alt="otel log collector" src="https://github.com/user-attachments/assets/8b7ee8ec-9b25-4fa2-b99d-75e41db75522" />


### Sending Test Metrics
```bash
curl -X POST http://localhost:4318/v1/metrics \
  -H "Content-Type: application/json" \
  -d '{
    "resourceMetrics": [{
      "resource": {
        "attributes": [{ "key": "service.name", "value": { "stringValue": "my-test-service" } }]
      },
      "scopeMetrics": [{
        "metrics": [{
          "name": "test_requests_total",
          "sum": {
            "dataPoints": [{
              "asInt": "42",
              "startTimeUnixNano": "1544712660000000000",
              "timeUnixNano": "1544712661000000000"
            }],
            "aggregationTemporality": 2,
            "isMonotonic": true
          }
        }]
      }]
    }]
  }'
```

Queried `test_requests_total` in Prometheus and confirmed the value.

**Data flow:** `curl` → OTEL Collector (OTLP receiver) → Prometheus exporter → Prometheus scrape. This is how OTEL bridges different telemetry formats into a common backend.

**Screenshot:** `test-request-total.png` — Prometheus successfully scraping metrics exported by the OpenTelemetry Collector.

<img width="1678" height="844" alt="tests request total" src="https://github.com/user-attachments/assets/9493020e-b934-4a26-b793-487beb6a8b5a" />

---

## Task 4 — Set Up Prometheus Alerting Rules

Prometheus evaluates alerting rules continuously and fires alerts when a PromQL condition stays true for a defined duration.

### `alert-rules.yml`
```yaml
groups:
  - name: system-alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage detected"
          description: "CPU usage has been above 80% for more than 2 minutes. Current value: {{ $value }}%"

      - alert: HighMemoryUsage
        expr: (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100 > 85
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage detected"
          description: "Memory usage is above 85%. Current value: {{ $value }}%"

      - alert: ContainerDown
        expr: absent(container_last_seen{name="notes-app"})
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Container is down"
          description: "The notes-app container has not been seen for over 1 minute"

      - alert: TargetDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Scrape target is down"
          description: "{{ $labels.job }} target {{ $labels.instance }} is unreachable"

      - alert: HighDiskUsage
        expr: (1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100 > 90
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Disk space running low"
          description: "Root filesystem usage is above 90%. Current value: {{ $value }}%"
```

### Rule Explanations

| Alert | Condition | Meaning |
|---|---|---|
| `HighCPUUsage` | Idle CPU < 20% for 2m | Sustained high CPU load |
| `HighMemoryUsage` | Memory usage > 85% for 2m | System running low on RAM |
| `ContainerDown` | `notes-app` container metric disappears for 1m | App container has stopped/crashed |
| `TargetDown` | Any scrape target reports `up == 0` for 1m | A monitored service is unreachable |
| `HighDiskUsage` | Root filesystem usage > 90% for 5m | Disk nearing capacity |

**Field meanings:**
- `expr` — the PromQL condition that triggers the alert
- `for` — how long the condition must hold true before firing (prevents flapping on brief spikes)
- `labels` — metadata used for routing (e.g., `severity`)
- `annotations` — human-readable alert description

### Wiring the Rules into Prometheus
```yaml
rule_files:
  - /etc/prometheus/alert-rules.yml
```

```yaml
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alert-rules.yml:/etc/prometheus/alert-rules.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    restart: unless-stopped
```

```bash
docker compose up -d prometheus
```

Verified all five rules under **Status → Rules**, and confirmed they were **inactive (green)** under **Alerts**.

**Screenshot:** `rules.png` — Prometheus successfully loaded all custom alert rules.

<img width="1672" height="574" alt="rules" src="https://github.com/user-attachments/assets/9f3f051e-b971-4525-8e04-07cfdf5484db" />


### Testing an Alert
```bash
docker compose stop notes-app
```
Waited ~1–2 minutes and watched `ContainerDown` move from **inactive → pending → firing**.

```bash
docker compose start notes-app
```

**Screenshot:** `container-down.png` — The `ContainerDown` alert entered the firing state after stopping the application container.

<img width="1680" height="470" alt="container down" src="https://github.com/user-attachments/assets/f231d2b1-04eb-4975-ac74-69d49ab91534" />

---

## Task 5 — Set Up Grafana Alerts

### Why Grafana Alerting?
Prometheus alerting rules fire and show state in its own UI, but **without Alertmanager, Prometheus cannot send notifications** (email, Slack, PagerDuty, etc.). Grafana has its own built-in alerting engine that can evaluate queries directly and route notifications to contact points, making it the simpler path to get real notifications flowing.

### Contact Point (SMTP)
- Went to **Alerting → Contact points → Add contact point**
- Name: `DevOps Team`
- Integration: **Email**, configured via SMTP settings
- Saved and sent a test notification to confirm delivery

### Alert Rule
- Went to **Alerting → Alert rules → New alert rule**
- Name: `High Container Memory`
- Query: `container_memory_usage_bytes{name="notes-app"} / 1024 / 1024`
- Condition: **IS ABOVE 100** (fires if the container uses more than 100MB)
- Evaluation: every `1m`, for `2m`
- Label: `severity = warning`
- Linked to the `DevOps Team` contact point

### Notification Policy
- Went to **Alerting → Notification policies**
- Set default contact point to `DevOps Team`
- Added a nested policy: `severity=critical` → routes to a separate/escalated contact point

Confirmed the alert rule state under **Alerting → Alert rules** (Normal / Pending / Firing), and confirmed the email notification arrived.

**Screenshot:** `email-received.png` — Grafana successfully sent an email notification through the configured SMTP server.

<img width="1307" height="745" alt="email received" src="https://github.com/user-attachments/assets/e65b4bed-95b4-49d2-b222-142c4ab8c220" />


**Screenshot:** `last-delivery-success.png` — Grafana Contact Point showing the latest notification delivery completed successfully.

<img width="1357" height="321" alt="last delivery attempt success" src="https://github.com/user-attachments/assets/0b8529fc-2d7c-4db2-b11b-158b9ac6d3d9" />




### Prometheus Alerts vs Grafana Alerts

| Aspect | Prometheus Alerting | Grafana Alerting |
|---|---|---|
| Evaluation | PromQL rules evaluated by Prometheus | Queries evaluated by Grafana against any connected data source |
| Notifications | Requires Alertmanager to route/send notifications | Built-in contact points (email, Slack, PagerDuty, etc.) — no extra component needed |
| Scope | Tied to Prometheus/PromQL only | Can alert across multiple data sources (Prometheus, Loki, etc.) |
| Best for | Infrastructure-level alerting close to the metrics engine, especially in setups that already run Alertmanager | Quick setup for notifications without deploying Alertmanager separately |

**When to use each:** Use Prometheus + Alertmanager when you want alerting fully decoupled from the visualization layer and already manage Alertmanager for routing/silencing at scale. Use Grafana alerting when you want a simpler, unified place to define alerts across multiple data sources and get notifications working quickly without standing up Alertmanager.

---

## Architecture Diagram

```
                        OBSERVABILITY STACK

                            METRICS
      Node Exporter ─────┐
      cAdvisor ──────────┼────► Prometheus ─────► Grafana Dashboards
      OTEL Collector ────┘              │
                                        ▼
                                 Alert Rules

  ------------------------------------------------------------

                              LOGS

     Docker Containers ─────► Promtail ─────► Loki ─────► Grafana Explore

  ------------------------------------------------------------

                             TRACES

     Application / curl ───► OTEL Collector ───► Debug Exporter
                                              │
                                              └──► Future: Jaeger / Grafana Tempo

  ------------------------------------------------------------

                         NOTIFICATIONS

                Prometheus / Grafana Alerts
                            │
                            ▼
                     Contact Point (SMTP)
                            │
                            ▼
                        Email Notifications
```

### Services Running

| Service | Port | Purpose |
|---|---|---|
| Prometheus | 9090 | Metrics storage and querying |
| Node Exporter | 9100 | Host system metrics |
| cAdvisor | 8080 | Container metrics |
| Grafana | 3000 | Visualization and alerting |
| Loki | 3100 | Log storage |
| Promtail | 9080 | Log collection agent |
| OTEL Collector | 4317 / 4318 / 8889 | Telemetry collection |
| Notes App | 8000 | Sample application |

```bash
docker compose ps
```
All 8 containers verified healthy and running.

**Screenshot:**

<img width="1330" height="192" alt="Screenshot 2026-08-06 at 10 40 34 PM" src="https://github.com/user-attachments/assets/2e359c55-1734-42c9-8379-a33750ad6979" />


---

## Folder Structure

```
day-76-otel-alerting/
├── otel-collector/
│   └── otel-collector-config.yml
├── alert-rules.yml
├── prometheus.yml
├── docker-compose.yml
├── day-76-otel-alerting.md
└── screenshots/
    ├── otel-up.png
    ├── otel-log-collector.png
    ├── test-request-total.png
    ├── rules.png
    ├── container-down.png
    ├── email-received.png
    └── last-delivery-success.png
```

---

## Commands Used

```bash
mkdir -p otel-collector
docker compose up -d
docker logs otel-collector 2>&1 | tail -5
docker logs otel-collector 2>&1 | grep -A 10 "test-span"
docker compose up -d prometheus
docker compose stop notes-app
docker compose start notes-app
docker compose ps
```

---

## Key Learnings

- OpenTelemetry is a **collection layer**, not a storage backend — it standardizes how telemetry is generated and shipped.
- The Collector's **receiver → processor → exporter** pipeline decouples data ingestion from data destination, so the same pipeline can fan out metrics, logs, and traces to different backends.
- **OTLP** is the common wire protocol (gRPC on 4317, HTTP on 4318) that lets any instrumented app talk to the Collector without vendor lock-in.
- Traces are made of **spans**, each representing one hop of a request across services — essential for debugging latency in distributed systems.
- Prometheus alerting rules fire based on `expr` + `for`, but need **Alertmanager** to actually notify anyone.
- **Grafana alerting** is the faster path to real notifications (email/Slack) since it has contact points and notification policies built in.
- A complete observability stack needs all **three pillars** — metrics, logs, traces — plus **alerting** to close the loop from "something broke" to "someone got notified."
