# Day 75 — Log Management with Loki and Promtail

Metrics tell you **what** is broken. Logs tell you **why**. Day 74 built the metrics pipeline — Prometheus, Node Exporter, cAdvisor, Grafana. Today adds the second pillar of observability: logs, via Grafana Loki (log storage) and Promtail (the agent that ships logs into it). By the end, Grafana shows metrics and logs side by side.

---

## Architecture

```text
                         Docker Host
                              |
              ┌───────────────┴───────────────┐
              |                               |
        Docker Containers                  cAdvisor
              |                               |
              | Logs                          | Metrics
              ▼                               ▼
          Promtail                       Prometheus
              |                               |
              ▼                               |
             Loki                             |
              |                               |
              └───────────────┬───────────────┘
                              ▼
                           Grafana
                              |
                    ┌─────────┴─────────┐
                    |                   |
                 Metrics               Logs
                    |                   |
                PromQL               LogQL
                    |                   |
                    └─────────┬─────────┘
                              ▼
                        Troubleshooting
```

| Component | Purpose |
|---|---|
| Prometheus | Stores and queries metrics |
| Node Exporter | Host-level metrics |
| cAdvisor | Container-level metrics |
| **Loki** | Stores and indexes logs |
| **Promtail** | Collects and ships logs to Loki |
| Grafana | Visualizes both metrics and logs |

---

## Task 1: Understand the Logging Pipeline

```text
[Docker Containers]
       |
       | write JSON logs to /var/lib/docker/containers/
       ▼
  [Promtail]
       |
       | reads log files, adds labels, pushes to Loki
       ▼
    [Loki]
       |
       | stores logs, indexes by labels only
       ▼
   [Grafana]
       |
       | queries Loki with LogQL, displays logs
       ▼
      [You]
```

### Why does Loki only index labels, not full text?

Traditional log systems like Elasticsearch build a full-text index over every word in every log message — powerful, but expensive to store and operate. Loki takes the opposite approach: it indexes only the **labels** attached to a log stream (e.g. `job="docker"`, `container_name="notes-app"`, `stream="stdout"`), and stores the actual log text as compressed chunks that get scanned at query time rather than pre-indexed.

It's essentially "Prometheus, but for logs" — same label-based mental model.

**The trade-off:**

| | Loki | Elasticsearch (ELK) |
|---|---|---|
| Indexing | Labels only | Full text of every message |
| Resource cost | Lower — smaller index, cheaper to run | Higher — large index, more storage/CPU |
| Architecture | Simple, pairs naturally with Prometheus/Grafana | More operational complexity |
| Search power | Weaker — line-by-line scan within a label-selected stream | Strong — arbitrary full-text search across everything |
| Best for | Correlating logs with metrics in a Grafana-centric stack | Deep, ad-hoc full-text log search at scale |

**Rule of thumb:** reach for Loki when logs are a companion to your metrics and you mostly filter by known dimensions (service, container, environment). Reach for Elasticsearch when you need to search unpredictable free text across huge log volumes as the primary use case.

---

## Task 2: Add Loki to the Stack

```bash
mkdir -p loki
```

### `loki/loki-config.yml`

```yaml
auth_enabled: false

server:
  http_listen_port: 3100

common:
  ring:
    instance_addr: 127.0.0.1
    kvstore:
      store: inmemory
  replication_factor: 1
  path_prefix: /loki

schema_config:
  configs:
    - from: 2020-10-24
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h

storage_config:
  filesystem:
    directory: /loki/chunks
```

| Setting | Meaning |
|---|---|
| `auth_enabled: false` | Single-tenant mode — no auth needed for a local learning setup |
| `http_listen_port: 3100` | Loki's HTTP API port |
| `replication_factor: 1` | Single instance, no replication (fine outside production) |
| `store: tsdb` | Uses Loki's time-series database for indexing |
| `object_store: filesystem` | Log chunks are stored on local disk |
| `path_prefix: /loki` | Loki's storage root path |

### Add to `docker-compose.yml`

```yaml
  loki:
    image: grafana/loki:latest
    container_name: loki
    ports:
      - "3100:3100"
    volumes:
      - ./loki/loki-config.yml:/etc/loki/loki-config.yml
      - loki_data:/loki
    command: -config.file=/etc/loki/loki-config.yml
    restart: unless-stopped
```

```yaml
volumes:
  prometheus_data:
  grafana_data:
  loki_data:
```

### Start and verify

```bash
docker compose up -d loki
curl http://localhost:3100/ready
```

Expected: `ready`

📸 **Verification Screenshot:** 

<img width="729" height="56" alt="3100 ready" src="https://github.com/user-attachments/assets/f9f8fda5-d8d1-4c4d-9fad-9915a4056054" />


---

## Task 3: Add Promtail to Collect Container Logs

Promtail is the log collection agent: it finds log files, tracks its read position, parses the Docker JSON log format, attaches labels, and pushes the result to Loki.

```bash
mkdir -p promtail
```

### `promtail/promtail-config.yml`

Using Docker service discovery so real container metadata (name, ID) gets attached as labels automatically:

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: docker

    docker_sd_configs:
      - host: unix:///var/run/docker.sock

    relabel_configs:
      - source_labels:
          - __meta_docker_container_name
        regex: "/(.*)"
        target_label: container_name

      - source_labels:
          - __meta_docker_container_id
        target_label: container_id

    pipeline_stages:
      - docker: {}
```

| Section | What it does |
|---|---|
| `positions` | Tracks how far Promtail has read into each log file — a bookmark, so a restart resumes rather than re-shipping everything |
| `clients` | Where logs get pushed — the Loki push API |
| `docker_sd_configs` | Discovers running Docker containers via the Docker socket, instead of hardcoding a file glob |
| `relabel_configs` | Converts Docker's internal metadata labels (`__meta_docker_container_name`) into a clean, queryable label (`container_name`) |
| `pipeline_stages: docker: {}` | Parses the Docker JSON log format and extracts timestamp, stream (stdout/stderr), and the actual message |

> A simpler static-path alternative (no Docker service discovery) just globs the log directory directly:
> ```yaml
> scrape_configs:
>   - job_name: docker
>     static_configs:
>       - targets: [localhost]
>         labels:
>           job: docker
>           __path__: /var/lib/docker/containers/*/*-json.log
>     pipeline_stages:
>       - docker: {}
> ```
> This works, but every log stream only gets labeled `job="docker"` — there's no `container_name` label to filter by individual container, which is why the service-discovery version above is the better setup for real querying.

### Add to `docker-compose.yml`

```yaml
  promtail:
    image: grafana/promtail:latest
    container_name: promtail
    volumes:
      - ./promtail/promtail-config.yml:/etc/promtail/promtail-config.yml
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock
    command: -config.file=/etc/promtail/promtail-config.yml
    restart: unless-stopped
```

**Why these mounts?**

| Mount | Purpose |
|---|---|
| `/var/lib/docker/containers` (read-only) | Where Docker writes container JSON log files |
| `/var/run/docker.sock` | Lets Promtail query Docker for container names/IDs via service discovery |

### Restart and generate logs

```bash
docker compose up -d

for i in $(seq 1 20); do curl -s http://localhost:8000 > /dev/null; done
```

### Verify Promtail is reading correctly

```bash
curl http://localhost:9080/targets
```

Expected: `docker (1/1 ready)`

📸 **Verification Screenshot:** 

<img width="3360" height="1750" alt="image" src="https://github.com/user-attachments/assets/97178669-9bc6-4e2d-98df-cb05454961d8" />

---

## Task 4: Add Loki as a Grafana Datasource

### Provision via YAML

`grafana/provisioning/datasources/datasources.yml`

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: false

  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    editable: false
```

```bash
docker compose restart grafana
```

Grafana now has two datasources side by side: **Prometheus** and **Loki**.

📸 **Verification Screenshot:** 

<img width="1672" height="713" alt="loki datasource running" src="https://github.com/user-attachments/assets/32da4ca7-6f8c-423a-a0cf-64104d3e3de0" />

---

## Task 5: Query Logs with LogQL

LogQL is Loki's query language — conceptually parallel to PromQL, but for log streams.

```logql
# All Docker logs
{job="docker"}

# Only the notes-app container (after Docker SD + relabeling)
{container_name="notes-app"}

# Keyword search — line contains "error" (case-sensitive)
{job="docker"} |= "error"

# Case-insensitive search
{job="docker"} |~ "(?i)error"

# Negative filter — exclude health-check noise
{job="docker"} != "health"

# Regex — HTTP 4xx/5xx status codes
{job="docker"} |~ "status=[45]\\d{2}"

# Count log lines over a 5-minute window
count_over_time({job="docker"}[5m])

# Per-second log rate
rate({job="docker"}[5m])

# Top 5 containers by log volume
topk(5, sum by (container_name) (rate({job="docker"}[5m])))
```

**Exercise — error logs for `notes-app` in the last hour, plus a per-minute error count:**

```logql
# Errors from notes-app, last 1h (set the time range picker to 1h)
{container_name="notes-app"} |= "error"

# Error lines per minute
count_over_time({container_name="notes-app"} |= "error" [1m])
```

### Label discovery, verified directly against Loki's API

```bash
curl -s http://localhost:3100/loki/api/v1/labels
```

Initially returned only: `filename`, `job`, `service_name`, `stream` — no `container_name` yet, since that label only appears once Docker service discovery and relabeling are configured.

```bash
curl -s http://localhost:3100/loki/api/v1/label/container_name/values
```

After adding `docker_sd_configs` and the relabeling rules, this returned:

```text
cadvisor
grafana
loki
node-exporter
notes-app
prometheus
promtail
```

Confirming Loki could now identify and filter by individual container name.

### Label cardinality — what makes a good label

| Good labels | Bad labels |
|---|---|
| `job`, `container_name`, `stream` | `user_id`, `request_id`, `transaction_id`, `timestamp` |

**Rule:** a label should identify a **log stream**, not an individual **log event**. High-cardinality values (unique per request) blow up Loki's index and defeat the whole point of the label-based design — that kind of detail belongs in the log line content, searched with `|=` or `|~`, not in a label.

---

## Task 6: Correlate Metrics and Logs in Grafana

This is the real payoff of Day 75: investigating an incident from **one interface** instead of switching between tools.

```text
                 Incident
                    |
             ┌──────┴──────┐
             |             |
          Metrics         Logs
             |             |
        Prometheus         Loki
             |             |
             └──────┬──────┘
                    |
                  Grafana
                    |
               Investigation
```

### Add a logs panel to the Day 74 dashboard

- Datasource: **Loki**
- Query: `{job="docker"}`
- Visualization: **Logs**
- Title: **"Container Logs"**

📸 **Verification Screenshot:** 

<img width="1556" height="903" alt="docker job" src="https://github.com/user-attachments/assets/416af08f-9241-4edb-9bad-6ad0415f5e2d" />


### Explore split view — metrics and logs side by side

**Left panel — Prometheus:**

```promql
rate(container_cpu_usage_seconds_total{id="/docker/<notes-app-container-id>"}[5m])
```

> Note: depending on how cAdvisor exposes container identity in your setup, the CPU metric may be keyed by container **ID** (`id="/docker/<hash>"`) rather than a clean `name="notes-app"` label. Either works — check `container_cpu_usage_seconds_total` in Prometheus to see which label your cAdvisor instance actually populates before writing the query.

**Right panel — Loki:**

```logql
{container_name="notes-app"}
```

Clicking a spike in the left (metrics) panel syncs the time range on the right (logs) panel — this is the core debugging workflow: see a metric anomaly, immediately check what the application was logging at that exact moment.

📸 **Verification Screenshot:** *Capture the Grafana Explore split view — Prometheus CPU metrics on the left, Loki notes-app logs on the right.*

<img width="1665" height="889" alt="split view" src="https://github.com/user-attachments/assets/9e3a98f5-4916-4665-b1fc-f6a954474909" />


### Why metrics + logs in one tool matters for incident response

A typical incident starts with a metric: CPU spikes, error rate climbs, latency jumps. But a metric only tells you *that* something is wrong — not *why*. Having Loki right next to Prometheus in Grafana means you can pivot from "CPU is high" to the actual log lines from that same container in that same time window — `Database connection failed`, `Request timeout`, `HTTP 500` — without leaving the tool, re-authenticating into a separate log system, or manually reconciling timestamps across two UIs. The time-range sync between panels is what turns "here's an anomaly" into "here's the root cause" in one motion instead of a multi-tool scavenger hunt.

---

## Complete Verification Commands

```bash
# Confirm all services are running
docker ps
# Expected: prometheus, grafana, loki, promtail, node-exporter, cadvisor, notes-app

# Loki health
curl http://localhost:3100/ready

# Promtail targets
curl http://localhost:9080/targets

# Loki's known labels
curl -s http://localhost:3100/loki/api/v1/labels

# Values for a specific label
curl -s http://localhost:3100/loki/api/v1/label/container_name/values

# Generate test traffic/logs
for i in $(seq 1 20); do curl -s http://localhost:8000 > /dev/null; done
```

---

## Key Takeaways

| Concept | Summary |
|---|---|
| **Loki** | Log aggregation system that indexes labels, not full text — cheaper and simpler than Elasticsearch, at the cost of full-text search power |
| **Promtail** | Agent that discovers containers, reads their logs, attaches labels, and ships everything to Loki |
| **LogQL** | Loki's query language — stream selectors (`{job="docker"}`), line filters (`|=`, `|~`, `!=`), and metric functions (`rate()`, `count_over_time()`) |
| **Docker service discovery** | Lets Promtail attach real container metadata (name, ID) as labels instead of a single flat `job` label |
| **Label cardinality** | Keep labels to low-cardinality dimensions (container, job, stream) — never per-request identifiers |
| **Correlation** | The actual value of observability isn't collecting metrics and logs separately — it's viewing them together, synced by time, in one tool |

**Day 74 → Day 75 shift:**

```text
Day 74:  Infrastructure Metrics
              ↓
         Prometheus + Node Exporter + cAdvisor → Grafana

Day 75:  Infrastructure Metrics + Logs
              ↓
         Prometheus + Loki + Promtail → Grafana
```

The stack now moves from simply *monitoring* infrastructure to being able to *investigate* incidents using correlated metrics and logs — setting up for Day 76's third pillar: distributed tracing with OpenTelemetry and Jaeger.
