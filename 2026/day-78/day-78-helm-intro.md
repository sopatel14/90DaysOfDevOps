# Day 78: Introduction to Kubernetes Package Management with Helm

## 1. Helm Concepts in My Own Words

Helm is the **package manager for Kubernetes** — the equivalent of `apt` for Ubuntu or `yum` for RHEL. Instead of writing, tracking, and applying dozens of raw manifest files by hand, Helm packages them into a single reusable, versioned unit and gives you a clean lifecycle (install / upgrade / rollback / uninstall) around it.

* **Chart**: A structured package containing all the template files needed to run an application — Deployments, Services, ConfigMaps, Secrets, PVCs, etc. — grouped into one folder along with metadata and default configuration. Think of it as the blueprint.
* **Release**: A specific running instance of a chart installed into a cluster. The same chart can be installed multiple times under different release names (e.g. `bankapp-mysql` and `bankapp-mysql-v2`), each tracked independently.
* **Repository**: A centralized location (like Docker Hub, but for charts) where packaged charts are published, versioned, and shared — e.g. Bitnami's chart repo.
* **Values**: The configuration layer (`values.yaml`) that acts as a parameter interface into the chart's templates. Instead of editing the underlying manifests directly, you override values like replica count, image tag, resource limits, or credentials.

### Why Helm over raw manifests?

Looking at the AI-BankApp's `k8s/` directory — 12 separate YAML files — every environment change means manually editing multiple files:

- **Templating**: one chart can serve dev, staging, and prod, each with its own values file, instead of duplicating and hand-editing YAML.
- **Versioning**: every change is tracked as a revision, so you can roll back instantly instead of doing a manual `git revert`.
- **Dependencies**: a chart can depend on other charts (e.g. the AI-BankApp chart could depend on a MySQL subchart).
- **Community**: thousands of production-ready charts already exist for common software — MySQL, Redis, Prometheus, ArgoCD — so you don't reinvent them.

---

## 2. Cluster Overview and Node Initialization

Before deploying anything, the underlying compute tier needs to be healthy and reachable.

```bash
kubectl cluster-info
kubectl get nodes -o wide
```

`> [INSERT SCREENSHOT: kubectl cluster-info and kubectl get nodes -o wide]`

```bash
helm version
helm list
```

<img width="1990" height="138" alt="image" src="https://github.com/user-attachments/assets/66d1e3e4-68d3-4238-80cf-7093de289d7a" />


---

## 3. Comparison: Raw Kubernetes YAML vs. Helm

| Aspect | Raw Manifests (`kubectl apply`) | Helm Chart Lifecycle (`helm install`) |
| :--- | :--- | :--- |
| **File management** | Track dozens of scattered YAML files (`deployment.yaml`, `pvc.yaml`, `secrets.yaml`...) | Bundles the whole application into one configurable package |
| **Environment parity** | Manually copy and hardcode values across dev/staging/prod files | One chart, reused across environments via separate `values.yaml` files |
| **Secrets** | Manually base64-encode and hardcode into secret manifests | Generated and managed automatically during install |
| **Storage** | Manual `StorageClass` + PVC files | Configured via a single `persistence.size` value |
| **Replicas** | Hardcoded in YAML | Controlled via a `replicaCount` value |
| **Metrics** | Usually not included, needs its own manifest | Toggle with `metrics.enabled: true` |
| **State/history** | No built-in history; manual `git revert` or file rewrites | Every change tracked as a numbered revision |
| **Rollback** | Manual re-apply of old YAML | Native `helm rollback` |

### AI-BankApp example command

```bash
helm install bankapp-mysql bitnami/mysql \
  --set auth.rootPassword=Test@123 \
  --set auth.database=bankappdb \
  --set primary.resources.requests.memory=256Mi \
  --set primary.resources.requests.cpu=250m \
  --set primary.resources.limits.memory=512Mi \
  --set primary.resources.limits.cpu=500m \
  --set primary.persistence.size=5Gi
```

One command replaces what would otherwise require `mysql-deployment.yml` + `secrets.yml` + `pvc.yml` + `pv.yml` + `service.yml` applied individually.

---

## 4. Custom Configuration: `mysql-values.yaml`

`--set` is fine for a couple of quick overrides, but anything beyond that belongs in a values file. Also, recent changes upstream in Bitnami's registry removed some community image tags from the public mainline repo, so this config explicitly points at the archived `bitnamilegacy` repository path to avoid image pull failures.

### File Content

```yaml
global:
  imageRegistry: docker.io
  imageRepository: bitnamilegacy

image:
  registry: docker.io
  repository: bitnamilegacy/mysql
  tag: 9.4.0

auth:
  rootPassword: Test@123
  database: bankappdb

primary:
  resources:
    limits:
      cpu: 500m
      memory: 512Mi
    requests:
      cpu: 250m
      memory: 256Mi
  persistence:
    size: 5Gi
    storageClass: ""

metrics:
  enabled: false
```

### Explanation of Each Field

* **`global`**: Overrides the chart's internal defaults so all baseline helper components (init containers, sidecars, etc.) also pull from the legacy image registry instead of the now-restricted mainline one.
* **`image`**: Pins the exact MySQL image and tag (`9.4.0`) so worker nodes don't hit random image pull failures from a moving/default tag.
* **`auth`**: Sets the root password and provisions the application's target schema database (`bankappdb`) automatically at first boot.
* **`primary.resources`**: Sets hard CPU/memory `requests` and `limits` so the pod has a predictable footprint on the cluster and doesn't starve other workloads.
* **`primary.persistence`**: Requests a 5Gi PVC for MySQL's data directory; leaving `storageClass` empty uses the cluster's default storage class.
* **`metrics.enabled`**: Toggles the Prometheus metrics exporter sidecar on/off — left `false` here, turned on later during the upgrade in Task 5.

---

## 5. Deploying and Verifying MySQL (Task 3 & 4)

Add the repository, then deploy using the values file:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo bitnami/mysql

helm install bankapp-mysql bitnami/mysql -f mysql-values.yaml
```

### Active Release Verification

```bash
helm list
kubectl get all -l app.kubernetes.io/instance=bankapp-mysql
kubectl get pvc -l app.kubernetes.io/instance=bankapp-mysql
kubectl get secret -l app.kubernetes.io/instance=bankapp-mysql
```

`> [INSERT SCREENSHOT: helm list showing bankapp-mysql release]`

### Live Database Check

```bash
kubectl exec -it bankapp-mysql-0 -- mysql -uroot -pTest@123 -e "SHOW DATABASES;"
```

Expected output includes `bankappdb` in the list of databases.

<img width="1736" height="498" alt="image" src="https://github.com/user-attachments/assets/27c0f71b-6f2b-4c28-b4f9-bbe33011e234" />


### Values File Reference and Cleanup (Task 4)

```bash
helm show values bitnami/mysql | head -80
```

This dumps every configurable knob the chart exposes — replication, custom init scripts, metrics, and more — all driven through values rather than editing templates.

```bash
helm install bankapp-mysql-v2 bitnami/mysql -f mysql-values.yaml
# ...verify it deployed alongside bankapp-mysql...
helm uninstall bankapp-mysql-v2
```


---

## 6. Release Management Lifecycle: Upgrade and Rollback (Task 5)

Helm tracks every operational change as an incremental, numbered revision, which is what makes rollback possible without manual `git revert` or YAML rewrites.

### Step 1: Upgrade — Enable Metrics

```bash
helm upgrade bankapp-mysql bitnami/mysql -f mysql-values.yaml \
  --set metrics.enabled=true \
  --set metrics.image.repository=bitnamilegacy/mysqld-exporter
```

### Step 2: Check Revision History

```bash
helm history bankapp-mysql
```

Expect to see revision 1 (original install) and revision 2 (metrics enabled).

### Step 3: Rollback to Revision 1

```bash
helm rollback bankapp-mysql 1
```

### Step 4: Verify History Again

```bash
helm history bankapp-mysql
```

Revision 3 now appears — a rollback *to* revision 1, not a deletion of history. Helm never erases past revisions; it always adds a new one.

<img width="1940" height="362" alt="image" src="https://github.com/user-attachments/assets/5743735b-672a-4983-8b4d-13542189b49c" />

**Comparison**: with raw `kubectl apply`, there is no built-in rollback — you'd have to `git revert` old manifests and re-apply them by hand. `helm rollback` gives you this out of the box, tied to the exact state of that revision.

---

## 7. Inside a Helm Chart: Internal Directory Structure (Task 6)

```bash
helm pull bitnami/mysql --untar
ls -R mysql/
```

```
mysql/
  Chart.yaml              # Chart metadata (name, version, description)
  values.yaml              # Default configuration values
  charts/                  # Subchart / dependency packages
  templates/                # Kubernetes manifest templates
    primary/
      statefulset.yaml      # StatefulSet template with Go template syntax
      svc.yaml               # Service template
    _helpers.tpl             # Reusable template helpers
    NOTES.txt                # Post-install message shown to the user
    secrets.yaml             # Secret template
```

<img width="592" height="222" alt="image" src="https://github.com/user-attachments/assets/2ef3c905-6cc3-47b5-9f87-aed2cbc3e6fc" />


### Hierarchy Breakdown

* **`Chart.yaml`**: Metadata hub — chart name, classification, dependencies, API version, and semantic version.
* **`values.yaml`**: The structural contract holding baseline/default parameters, used whenever a custom values file doesn't override them.
* **`templates/`**: The actual Kubernetes manifests, written with Go templating syntax so values get injected dynamically, e.g.:
  ```yaml
  replicas: {{ .Values.primary.replicaCount }}
  image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
  ```
  Passing `--set primary.replicaCount=3` overrides `{{ .Values.primary.replicaCount }}` directly — no manifest editing required.
* **`charts/`**: A directory for nesting standalone dependency (sub)charts that the primary chart relies on.

### Key Distinction: `version` vs. `appVersion`

Looking at `Chart.yaml`:

```yaml
apiVersion: v2
name: mysql
description: A Helm chart for MySQL
version: 12.2.1      # Chart version
appVersion: "9.4.0"   # Version of MySQL running inside the chart
```

* **`version`**: The semantic version of the **Helm chart itself** — bumped when the chart's structure, templates, helpers, or default values change.
* **`appVersion`**: The version of the **actual application/binary** the chart deploys — in this case, the MySQL server version running inside the container.

They're independent: a chart can bump `version` (e.g. fixing a template bug) without changing which MySQL `appVersion` it ships, and vice versa.

### Cleanup

```bash
helm uninstall bankapp-mysql
rm -rf mysql/
```

---

## 8. Why the AI-BankApp's 12 Manifests Benefit from Helm Migration

The AI-BankApp currently ships 12 raw YAML files in `k8s/` — `bankapp-deployment.yml`, `configmap.yml`, `gateway.yml`, `mysql-deployment.yml`, `namespace.yml`, `ollama-deployment.yml`, `pv.yml`, `pvc.yml`, `secrets.yml`, `service.yml`, `hpa.yml`, `cert-manager.yml`. Migrating this to a Helm chart solves several real bottlenecks:

1. **Single Command Deployments**: Replaces a chain of individual `kubectl apply -f ...` calls with one `helm install` / `helm upgrade --install`.
2. **Dynamic Component Interconnectivity**: Values can flow between components — e.g. the MySQL secret Helm generates can be fed straight into `bankapp-deployment.yml`'s environment variables — instead of hardcoding credentials in multiple places.
3. **Environment Parity**: The same chart can serve dev/staging/prod by swapping out a `values-<env>.yaml` file, instead of maintaining near-duplicate manifest sets per environment.
4. **Built-in Rollback & History**: Every change becomes a tracked revision, so a bad deploy can be rolled back with `helm rollback` instead of a manual `git revert` and re-apply.
5. **Cluster Safety Assurance**: Decouples configuration from environment-specific constants, reducing the risk of credentials or config drift being hardcoded into version-controlled manifests.

This is exactly what tomorrow's task (Day 79) builds: converting these 12 raw manifests into a proper Helm chart for AI-BankApp.
