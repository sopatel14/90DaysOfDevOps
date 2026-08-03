# Day 73 — Introduction to Observability and Prometheus

You've built infrastructure with Terraform, configured it with Ansible, and containerized apps with Docker. But once everything's running — how do you know it's healthy, and how do you find out why something broke at 3 AM? That's what observability answers. Today: the three pillars of observability, and standing up Prometheus — the most widely used metrics tool in the DevOps ecosystem.

---

## Task 1: Understanding Observability

### What is Observability?

Observability is the ability to understand what's happening *inside* a system just by looking at what it outputs — its metrics, logs, and traces. Instead of asking pre-defined questions, it lets you explore and query freely to figure out things you didn't think to monitor in advance.

### Monitoring vs Observability

| Aspect | Monitoring | Observability |
|---|---|---|
| Purpose | Tells you **what** is wrong | Tells you **why** it's wrong |
| Approach | Pre-defined metrics and alerts | Explore and query data freely |
| Typical question | "Is the server down?" | "Why is the API slow?" |
| Data scope | Limited to what you thought to monitor ahead of time | All data, accessible for open-ended exploration |

In short: monitoring is reactive and threshold-based ("CPU > 90%, alert me"). Observability is investigative — it gives you the raw material to answer questions you never anticipated.

### The Three Pillars

**1. Metrics** — numerical data points collected over time.
- Examples: CPU usage, memory, request count, error rate
- Tools: Prometheus, Datadog, CloudWatch

**2. Logs** — timestamped text records of individual events.
- Examples: application logs, error messages, access logs
- Tools: Loki, ELK Stack, Fluentd

**3. Traces** — the journey of a single request as it moves through multiple services.
- Example: User request → API → Database → Response
- Tools: OpenTelemetry, Jaeger, Zipkin

### Why DevOps Engineers Need All Three

A real incident usually needs all three pillars working together:

1. **Metrics** show a high error rate on the payment service.
2. **Logs** reveal the actual cause: `"Database connection timeout"`.
3. **Traces** show exactly where the time went: the payment service call took 12 seconds because of a slow database query.

Metrics tell you **what** is broken, logs tell you **why**, and traces tell you **where**.

### Architecture — What Gets Built Over Days 73–77

```text
[Your App] ──metrics──▶ [Prometheus]      ──▶ [Grafana Dashboards]
[Your App] ──logs────▶  [Promtail]  ──▶ [Loki] ──▶ [Grafana]
[Your App] ──traces───▶ [OTEL Collector] ──▶ [Grafana / Jaeger]
[Host]     ──metrics──▶ [Node Exporter]   ──▶ [Prometheus]
[Docker]   ──metrics──▶ [cAdvisor]        ──▶ [Prometheus]
```

Everything ultimately lands in Grafana as a single pane of glass over metrics, logs, and traces.

---

## Task 2: Set Up Prometheus with Docker

Created a dedicated project directory to build on across the whole observability block:

```bash
mkdir observability-stack && cd observability-stack
```

### `prometheus.yml`

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
```

This tells Prometheus to scrape its own `/metrics` endpoint every 15 seconds.

### `docker-compose.yml`

```yaml
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    restart: unless-stopped

volumes:
  prometheus_data:
```

### Start it

```bash
docker compose up -d
```

Open `http://localhost:9090` — the Prometheus web UI should load.

**Verify:** go to **Status → Targets**. There should be one target (`prometheus`) with state **UP**.

📸 **Verification Screenshot:** 

<img width="1678" height="230" alt="image" src="https://github.com/user-attachments/assets/c9290ff4-5977-47cf-81b1-4e75d6bf2fc5" />


---

## Task 3: Prometheus Concepts

- **Scrape targets** — endpoints Prometheus pulls metrics from at regular intervals (a pull-based model, not push-based).
- **Labels** — key-value pairs that add dimensions to a metric, e.g. `http_requests_total{method="GET", status="200"}`.
- **Time series** — a unique combination of metric name + label set. `{method="GET"}` and `{method="POST"}` on the same metric name are two different time series.

### Metric Types

| Type | Behavior | Example |
|---|---|---|
| **Counter** | Only ever increases | Total requests served, total errors |
| **Gauge** | Goes up *and* down | Current CPU usage, active connections, memory in use |
| **Histogram** | Distribution of values across buckets | Request duration: how many took <100ms, <500ms, <1s |
| **Summary** | Like a histogram, but percentiles are calculated client-side | Client-computed request latency percentiles |

**Counter vs Gauge — the key distinction:**
A counter never resets except on restart (it only climbs), so it's meaningful with `rate()` to get a speed. A gauge is a snapshot value that can rise or fall freely, so `rate()` on it produces meaningless results.

- **Real-world counter example:** `http_requests_total` — the running total of HTTP requests ever served.
- **Real-world gauge example:** `process_resident_memory_bytes` or current active DB connections — reflects the value *right now*, which can go up or down.

### Queries Run in the UI

```promql
# How many metrics is Prometheus collecting about itself?
count({__name__=~".+"})

# How much memory is Prometheus using?
process_resident_memory_bytes

# Total HTTP requests to the Prometheus server
prometheus_http_requests_total

# Break it down by handler
prometheus_http_requests_total{handler="/api/v1/query"}
```

📸 **Verification Screenshot:** *Capture the Prometheus Graph page showing one of the queries above with results.*

<img width="1680" height="841" alt="Task 2 - 1" src="https://github.com/user-attachments/assets/05a743a0-2fa2-43f7-96e3-9593abd4e989" />

<img width="1661" height="823" alt="Task 2 -2" src="https://github.com/user-attachments/assets/a49ff642-f890-4e77-ad6b-e01b34948e73" />

<img width="1678" height="831" alt="Task 2 -3" src="https://github.com/user-attachments/assets/1c867989-69a6-40d7-aa9c-cd0ee128db4d" />

<img width="1670" height="839" alt="Task 2 -4" src="https://github.com/user-attachments/assets/436437d9-f6d3-41d7-b0bb-b6cd87518dba" />

<img width="1680" height="842" alt="Task 2 -5" src="https://github.com/user-attachments/assets/aef22fec-606a-42d0-8917-f3b8ef01479e" />

---

## Task 4: PromQL Basics

| Query type | Example | What it means |
|---|---|---|
| Instant vector | `up` | Current value per target — `1` = up, `0` = down |
| Range vector | `prometheus_http_requests_total[5m]` | All raw values from the last 5 minutes |
| Rate | `rate(prometheus_http_requests_total[5m])` | Per-second rate of a counter over a window — the most-used function |
| Aggregation | `sum(rate(prometheus_http_requests_total[5m]))` | Sums the rate across all label combinations |
| Label filter | `prometheus_http_requests_total{code="200"}` | Only series matching that label value |
| Negative filter | `prometheus_http_requests_total{code!="200"}` | Excludes that label value |
| Arithmetic | `process_resident_memory_bytes / 1024 / 1024` | Converts bytes → megabytes |
| Top-K | `topk(5, prometheus_http_requests_total)` | Highest 5 series by value |

### Exercise: Per-Second Rate of Non-200 Requests

```promql
rate(prometheus_http_requests_total{code!="200"}[5m])
```

This filters to only non-200 responses first, then computes their per-second rate over a 5-minute window — exactly the pattern you'd use to alert on a rising error rate.

📸 **Verification Screenshot:** 


> **Important ordering rule:** always `sum(rate(...))`, never `rate(sum(...))`. Rate only makes sense applied directly to a raw counter; summing first destroys the per-series counter resets that `rate()` needs to detect correctly.

<img width="1680" height="755" alt="image" src="https://github.com/user-attachments/assets/eb8ab104-da68-4c67-9913-5818b90a87e3" />


---

## Task 5: Add a Sample Application as a Scrape Target

### Updated `docker-compose.yml`

```yaml
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    restart: unless-stopped

  notes-app:
    image: trainwithshubham/notes-app:latest
    container_name: notes-app
    ports:
      - "8000:8000"
    restart: unless-stopped

volumes:
  prometheus_data:
```

### Updated `prometheus.yml`

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "notes-app"
    static_configs:
      - targets: ["notes-app:8000"]
```

### Restart and generate traffic

```bash
docker compose up -d

curl http://localhost:8000
curl http://localhost:8000/health
curl http://localhost:8000/api/notes
```

**Verify:** **Status → Targets** now shows two healthy targets — `prometheus` and `notes-app`.

📸 **Verification Screenshot:** *Capture the Targets page showing both `prometheus` and `notes-app` as UP.*

![Prometheus Targets — With Notes App](path/to/your/screenshot.png)

> Not every application exposes Prometheus metrics natively. Node Exporter, cAdvisor, and the OTEL Collector (covered on Days 74–76) act as exporters for systems that don't have built-in Prometheus support.

---

## Task 6: Data Retention and Storage

### Check disk usage

```bash
docker exec prometheus du -sh /prometheus
```

### Configure retention

```yaml
command:
  - '--config.file=/etc/prometheus/prometheus.yml'
  - '--storage.tsdb.retention.time=30d'
  - '--storage.tsdb.retention.size=5GB'
```


📸 **Verification Screenshot:** 

<img width="3358" height="1812" alt="image" src="https://github.com/user-attachments/assets/a3eb4524-e789-48b9-a012-004fa2a0a491" />


### What happens when retention is exceeded?

Prometheus's local TSDB isn't infinite — once data is older than the configured `retention.time` (default 15 days) or the store exceeds `retention.size`, the oldest blocks are compacted and deleted automatically to free space. Older data simply becomes unqueryable once it's dropped; there's no built-in archival, which is why long-term storage setups typically pair Prometheus with remote-write to a system like Thanos or Cortex.

### Why is a volume mount important?

The `prometheus_data:/prometheus` volume mount persists the TSDB outside the container's writable layer. Without it, `docker compose down` (or any container recreation) wipes all collected metrics history, since container filesystems are ephemeral by default. The named volume keeps the time-series data intact across restarts, upgrades, and redeploys.

---

## Common Commands Reference

```bash
# Start the stack
docker compose up -d

# Stop the stack
docker compose down

# Check logs
docker compose logs -f prometheus

# Check storage usage
docker exec prometheus du -sh /prometheus

# Generate traffic for the notes app
curl http://localhost:8000
curl http://localhost:8000/health
curl http://localhost:8000/api/notes
```

---

## Learning Summary

- **Observability basics** — the three pillars (metrics, logs, traces) and why monitoring alone isn't enough.
- **Prometheus setup** — running it via Docker and Docker Compose with a mounted config and persistent volume.
- **Prometheus concepts** — counters vs gauges vs histograms, labels, and time series.
- **PromQL** — instant vectors, range vectors, `rate()`, aggregation, label filters, and top-K.
- **Sample application** — added `notes-app` as a second scrape target and generated real traffic.
- **Data management** — retention limits, TSDB behavior, and why the volume mount matters.

## Architecture diagram of what will be build over days 73-77

┌─────────────────────────────────────────────────────────────────┐
│                     Observability Stack                        │
│                                                               │
│  ┌──────────┐    ┌──────────┐    ┌────────────────────────┐  │
│  │ Your App │───▶│ Prometheus│───▶│  Grafana Dashboards   │  │
│  └──────────┘    └──────────┘    └────────────────────────┘  │
│                                                               │
│  ┌──────────┐    ┌──────────┐    ┌────────────────────────┐  │
│  │ Your App │───▶│ Promtail │───▶│  Loki (Logs)          │  │
│  └──────────┘    └──────────┘    └────────────────────────┘  │
│                                                               │
│  ┌──────────┐    ┌──────────┐    ┌────────────────────────┐  │
│  │ Your App │───▶│ OTEL     │───▶│  Jaeger (Traces)      │  │
│  └──────────┘    │ Collector│    └────────────────────────┘  │
│                   └──────────┘                                │
│                                                               │
│  ┌──────────┐    ┌──────────┐                                │
│  │ Host     │───▶│ Node     │                                │
│  │ Metrics  │    │ Exporter │                                │
│  └──────────┘    └──────────┘                                │
│                                                               │
│  ┌──────────┐    ┌──────────┐                                │
│  │ Docker   │───▶│ cAdvisor │                                │
│  │ Metrics  │    │          │                                │
│  └──────────┘    └──────────┘                                │
│                                                               │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │        All data visualized in Grafana                  │ │
│  └──────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘

### Next Steps (Days 74–77)

- **Day 74:** Node Exporter and cAdvisor
- **Day 75:** Loki and Promtail for logs
- **Day 76:** OpenTelemetry and Jaeger for traces
- **Day 77:** Grafana dashboards
