# Day 74 — Node Exporter, cAdvisor & Grafana Dashboards

Prometheus is running and answering PromQL queries — but so far it's only monitoring itself. In a real setup you need to watch two other critical things: the **host machine** (CPU, memory, disk, network) and the **Docker containers** running on it. Today adds Node Exporter for host metrics, cAdvisor for container metrics, and Grafana to turn all of it into dashboards instead of raw PromQL.

---

## Architecture

```text
                         ┌─────────────────────┐
                         │       Grafana        │
                         │        :3000         │
                         └──────────┬───────────┘
                                    │
                                  PromQL
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │      Prometheus      │
                         │        :9090         │
                         └──────────┬───────────┘
                                    │
                  ┌─────────────────┼─────────────────┐
                  │                 │                 │
                  ▼                 ▼                 ▼
          ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
          │Node Exporter │  │   cAdvisor   │  │  Prometheus  │
          │    :9100     │  │    :8080     │  │    :9090     │
          └──────┬───────┘  └──────┬───────┘  └──────────────┘
                 │                 │
                 ▼                 ▼
           Host Metrics       Container Metrics
```

By the end of today, the stack contains: **Prometheus, Node Exporter, cAdvisor, Grafana**, and the notes app carried over from Day 73.

---

## Task 1: Node Exporter — Host Metrics

### Theory

Node Exporter is a Prometheus exporter that exposes host-level operating system metrics in Prometheus format — CPU, memory, disk, filesystem, network, system load, and general system info. It answers one question: **"How healthy is my host machine?"**

All of its metrics are prefixed `node_`:

```text
node_cpu_seconds_total
node_memory_MemTotal_bytes
node_memory_MemAvailable_bytes
node_filesystem_size_bytes
node_network_receive_bytes_total
```

### Add to `docker-compose.yml`

```yaml
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--path.rootfs=/rootfs'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    restart: unless-stopped
```

### Why these volume mounts?

| Mount | Purpose |
|---|---|
| `/proc` | CPU, memory, and process information |
| `/sys` | Kernel, hardware, and driver information |
| `/` | Filesystem and disk usage |

All three are mounted **read-only** (`:ro`) — Node Exporter only ever reads system information, never modifies it.

### Add as a scrape target — `prometheus.yml`

```yaml
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node-exporter"
    static_configs:
      - targets: ["node-exporter:9100"]
```

### Restart and verify

```bash
docker compose up -d
curl http://localhost:9100/metrics | head -20
```

Check **Status → Targets** — `node-exporter` should show **UP**.

### Host metric queries

```promql
# CPU: time spent idle (per core)
node_cpu_seconds_total{mode="idle"}

# Memory: total vs available
node_memory_MemTotal_bytes
node_memory_MemAvailable_bytes

# Memory usage percentage
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100

# Disk: filesystem usage percentage
(1 - node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100

# Network: bytes received per second
rate(node_network_receive_bytes_total[5m])
```

---

## Task 2: cAdvisor — Container Metrics

### Theory

cAdvisor (Container Advisor) monitors resource usage and performance at the **container** level rather than the host level. It answers: **"Which container is using my resources?"**

Its metrics are prefixed `container_`:

```text
container_cpu_usage_seconds_total
container_memory_usage_bytes
container_network_receive_bytes_total
```

### Add to `docker-compose.yml`

```yaml
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    restart: unless-stopped
```

### Why these volume mounts?

| Mount | Purpose |
|---|---|
| `/var/run/docker.sock` | Lets cAdvisor discover and inspect running containers |
| `/sys` | Kernel-level cgroup stats |
| `/var/lib/docker/` | Container filesystem information |

### Add as a scrape target

```yaml
  - job_name: "cadvisor"
    static_configs:
      - targets: ["cadvisor:8080"]
```

### Restart and verify

```bash
docker compose up -d
```

Open `http://localhost:8080` → **Docker Containers** to see per-container stats in cAdvisor's own web UI.

📸 **Verification Screenshot:** 

<img width="1576" height="894" alt="Task 2 - ca advisor" src="https://github.com/user-attachments/assets/f43f17cf-d0e9-437f-8d55-4780265c5ffb" />




### Container metric queries

```promql
# CPU usage per container
rate(container_cpu_usage_seconds_total{name!=""}[5m])

# Memory usage per container
container_memory_usage_bytes{name!=""}

# Network received bytes per container
rate(container_network_receive_bytes_total{name!=""}[5m])

# Top 3 memory-consuming containers
topk(3, container_memory_usage_bytes{name!=""})
```

The `{name!=""}` filter excludes aggregated/system-level entries that don't correspond to a named container.

### Node Exporter vs cAdvisor

| Feature | Node Exporter | cAdvisor |
|---|---|---|
| Monitoring level | Host | Container |
| CPU | Host CPU | Container CPU |
| Memory | Host memory | Container memory |
| Disk | Host filesystem | Container filesystem |
| Network | Host network | Container network |
| Metric prefix | `node_` | `container_` |
| Typical use | VM/server monitoring | Docker/container monitoring |

**When to use Node Exporter:** diagnosing host-level issues — high CPU load, low available memory, disk almost full, high network traffic, system load problems.

**When to use cAdvisor:** diagnosing per-container issues — which container is eating CPU or memory, which one is generating network traffic, which one is becoming unhealthy.

In a containerized environment you generally want **both**: Node Exporter for "is the machine okay," cAdvisor for "which container is the problem."

### Prometheus Targets check

```bash
docker compose ps
```

Expected running services: `prometheus`, `notes-app`, `node-exporter`, `cadvisor`.

📸 **Verification Screenshot:** 

<img width="3336" height="1332" alt="image" src="https://github.com/user-attachments/assets/6e2a22a1-11f3-425b-b088-fd91e0ff1805" />


---

## Task 3: Set Up Grafana

### Theory

Prometheus collects, stores, and queries metrics. Grafana is the **visualization layer** on top: it turns PromQL results into dashboards, gauges, and charts, and provides a shared view the whole team can look at instead of everyone running raw queries.

### Add to `docker-compose.yml`

```yaml
  grafana:
    image: grafana/grafana-enterprise:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    restart: unless-stopped
```

Add the volume:

```yaml
volumes:
  prometheus_data:
  grafana_data:
```

### Restart and log in

```bash
docker compose up -d
```

Open `http://localhost:3000` and log in with `admin` / `admin123`.

📸 **Verification Screenshot:** 

<img width="3260" height="1794" alt="image" src="https://github.com/user-attachments/assets/83b35287-38ce-4c1f-a0d5-c40c1e3c0f42" />


### Add Prometheus as a datasource

1. **Connections → Data Sources → Add data source**
2. Select **Prometheus**
3. Set URL to `http://prometheus:9090`
4. **Save & Test** → should report *"Successfully queried the Prometheus API"*

**Why `http://prometheus:9090` and not `localhost:9090`?** Grafana and Prometheus run in separate containers. Inside the Grafana container, `localhost` refers to the Grafana container itself — not Prometheus. Docker Compose provides internal DNS, so containers reach each other by **service name**, hence `prometheus:9090`.

```text
Grafana ──http://prometheus:9090──▶ Prometheus
```

📸 **Verification Screenshot:** 

<img width="3354" height="1108" alt="image" src="https://github.com/user-attachments/assets/e157f4f3-bdf7-4a7c-9a21-f2170733723f" />


---

## Task 4: Build the First Dashboard

Created **DevOps Observability Overview** — a dashboard with five panels covering both host and container health.

| Panel | PromQL | Visualization |
|---|---|---|
| CPU Usage % | `100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)` | Gauge (green <60, yellow <80, red ≥80) |
| Memory Usage % | `(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100` | Gauge |
| Container CPU Usage | `rate(container_cpu_usage_seconds_total{name!=""}[5m]) * 100` | Time series, legend `{{name}}` |
| Container Memory (MB) | `container_memory_usage_bytes{name!=""} / 1024 / 1024` | Bar chart, legend `{{name}}` |
| Disk Usage % | `(1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100` | Stat |

**Query notes:**
- The CPU query works by taking the idle-time rate and subtracting it from 100 — what's left is "busy" time.
- `container_*` queries all filter on `{name!=""}` to skip cAdvisor's aggregated/root cgroup entries and only show real containers.

📸 **Verification Screenshot:** *Capture the full "DevOps Observability Overview" 

<img width="3302" height="1760" alt="image" src="https://github.com/user-attachments/assets/e79f1b0e-5a6f-4472-bffd-86602d2223b6" />


---

## Task 5: Auto-Provision Datasources with YAML

### Theory

Clicking through the Grafana UI to add a datasource works, but it isn't repeatable — a fresh environment means doing it all over again by hand. **Provisioning** lets Grafana load datasource (and dashboard) configuration automatically from YAML files at startup.

### Directory structure

```bash
mkdir -p grafana/provisioning/datasources
mkdir -p grafana/provisioning/dashboards
```

### `grafana/provisioning/datasources/datasources.yml`

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: false
```

### Mount the provisioning directory

```yaml
  grafana:
    image: grafana/grafana-enterprise:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    restart: unless-stopped
```

```bash
docker compose up -d grafana
```

**Connections → Data Sources** now shows Prometheus already configured — with zero manual clicking.

### Why YAML provisioning beats manual configuration

```text
Manual                          Provisioned
Open Grafana                    Git repository
  ↓                                ↓
Add datasource                  datasources.yml
  ↓                                ↓
Enter Prometheus URL            Grafana starts
  ↓                                ↓
Save                             Datasource auto-configured
```

- **Repeatable** — spin up a fresh environment and it's configured identically every time.
- **Version-controlled** — changes go through Git and code review, same as application code.
- **Automated / CI-CD friendly** — no manual steps blocking a deploy pipeline.
- **Reduces configuration drift** — every environment reads from the same source of truth instead of accumulating manual differences.

This is the "configuration as code" principle: infrastructure and application config live and get reviewed alongside the project source, not hidden in someone's browser session.

---

## Task 6: Import a Community Dashboard

Rather than building every panel from scratch, Grafana's community library has thousands of pre-built dashboards.

**Node Exporter Full** — Dashboard ID `1860`:

1. **Dashboards → New → Import**
2. Enter dashboard ID `1860`
3. Select the Prometheus datasource
4. **Import**

This single dashboard covers CPU, memory, disk, filesystem, network, load, and system info in far more depth than the custom panels built in Task 4 — it's considered close to the standard for host monitoring.

📸 **Verification Screenshot:** 

<img width="3154" height="1704" alt="image" src="https://github.com/user-attachments/assets/1db859cc-843c-4698-b7d7-9248a0b93131" />


**Docker / cAdvisor** — Dashboard ID `193` was also imported for container-level stats, same process, Prometheus as the datasource.

### Final service check

```bash
docker compose ps
```

Expected services: `prometheus`, `node-exporter`, `cadvisor`, `grafana`, `notes-app`.

| Service | URL |
|---|---|
| Prometheus | http://localhost:9090 |
| Node Exporter | http://localhost:9100/metrics |
| cAdvisor | http://localhost:8080 |
| Grafana | http://localhost:3000 |
| Notes App | http://localhost:8000 |

---

## Complete `docker-compose.yml`

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

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--path.rootfs=/rootfs'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    restart: unless-stopped

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    restart: unless-stopped

  grafana:
    image: grafana/grafana-enterprise:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    restart: unless-stopped

volumes:
  prometheus_data:
  grafana_data:
```

## Complete `prometheus.yml`

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:

  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node-exporter"
    static_configs:
      - targets: ["node-exporter:9100"]

  - job_name: "cadvisor"
    static_configs:
      - targets: ["cadvisor:8080"]
```

## Grafana Datasource Provisioning File

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
```

---

## Final Architecture

```text
                          ┌──────────────────────┐
                          │        Grafana        │
                          │         :3000         │
                          │                        │
                          │   Custom Dashboard     │
                          │   Node Exporter #1860  │
                          └──────────┬─────────────┘
                                     │
                                   PromQL
                                     │
                                     ▼
                          ┌──────────────────────┐
                          │      Prometheus       │
                          │         :9090         │
                          │                        │
                          │    Time-Series DB      │
                          └──────────┬─────────────┘
                                     │
                ┌────────────────────┼────────────────────┐
                │                    │                    │
                ▼                    ▼                    ▼
        ┌───────────────┐    ┌───────────────┐    ┌───────────────┐
        │ Node Exporter │    │    cAdvisor   │    │   Prometheus  │
        │     :9100     │    │     :8080     │    │     :9090     │
        └───────┬───────┘    └───────┬───────┘    └───────────────┘
                │                    │
                ▼                    ▼
          Host Metrics         Container Metrics
     CPU / Memory / Disk    CPU / Memory / Network
```

---

## Key Takeaways

| Component | Role |
|---|---|
| **Prometheus** | Collects, stores, and queries time-series metrics via PromQL |
| **Node Exporter** | Host-level OS metrics (`node_*`) — CPU, memory, disk, network |
| **cAdvisor** | Container-level resource metrics (`container_*`) |
| **Grafana** | Turns Prometheus data into dashboards and visualizations |
| **Provisioning** | Datasources (and dashboards) defined as YAML, loaded automatically on startup |
| **Community dashboards** | Pre-built, battle-tested visualizations (e.g. Node Exporter Full #1860) imported instead of hand-built |

A useful observability system needs to do all five things — **collect, store, query, visualize, and stay repeatable** — not just gather numbers. With Node Exporter, cAdvisor, Prometheus, and Grafana working together, the stack now has a solid base for what's next: alerting with Alertmanager, centralized logging with Loki, distributed tracing, and SLIs/SLOs.
