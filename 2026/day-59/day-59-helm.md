# Day 59: Helm - Kubernetes Package Manager

---

## Task 1: Install Helm

### What is Helm?

Helm is the package manager for Kubernetes, similar to `apt` for Ubuntu or `brew` for macOS. It helps you define, install, and upgrade even the most complex Kubernetes applications.

### Installation Verification

```bash
souravpatel@Souravs-MacBook-Pro kuberenetes-practice % helm version
version.BuildInfo{Version:"v4.2.3", GitCommit:"43e8b7feece8beb0fcba47059ec9b522fd929a64", GitTreeState:"clean", GoVersion:"go1.26.5", KubeClientVersion:"v1.36"}
```

| Property | Value |
|---|---|
| Helm Version Installed | v4.2.3 |
| Git Commit | 43e8b7feece8beb0fcba47059ec9b522fd929a64 |
| Go Version | go1.26.5 |
| Kubernetes Client Version | v1.36 |

### Helm Environment Variables

```text
HELM_BIN="helm"
HELM_CACHE_HOME="/Users/souravpatel/Library/Caches/helm"
HELM_CONFIG_HOME="/Users/souravpatel/Library/Preferences/helm"
HELM_DATA_HOME="/Users/souravpatel/Library/helm"
HELM_NAMESPACE="default"
HELM_MAX_HISTORY="10"
```

### Three Core Concepts

- **Chart**: A package of Kubernetes manifest templates
  - Contains all YAML files needed to deploy an application
  - Includes templates for Deployments, Services, ConfigMaps, etc.
- **Release**: A specific installation of a chart in your cluster
  - Each installation creates a unique release
  - Can have multiple releases of the same chart (e.g., dev, staging, prod)
- **Repository**: A collection of charts (like a package repo)
  - Public repos: Bitnami, Stable, etc.
  - Can host private repositories

> 📸 **Screenshot: `helm version` output**

<img width="2074" height="918" alt="image" src="https://github.com/user-attachments/assets/74172dc7-f03b-4221-9160-0030d0ee1c13" />


---

## Task 2: Add Repository and Search

### Repository Management Commands

```bash
# Add Bitnami repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# Update repository cache
helm repo update
```

### Search Results Analysis

```bash
souravpatel@Souravs-MacBook-Pro kuberenetes-practice % helm search repo bitnami | wc -l
     145
```

**Total Charts in Bitnami Repository:** 145 charts (including the header line, so 144 actual charts)

### Example Charts Found

- `bitnami/nginx` - Nginx web server
- `bitnami/mysql` - MySQL database
- `bitnami/postgresql` - PostgreSQL database
- `bitnami/redis` - Redis cache
- `bitnami/kafka` - Apache Kafka
- And many more...

> 📸 **Screenshot: `helm repo update` output**

<img width="632" height="96" alt="image" src="https://github.com/user-attachments/assets/40f36163-7c0b-4b34-a9ae-950c011fc76a" />


> 📸 **Screenshot: `helm search repo bitnami | wc -l` output**

<img width="1814" height="118" alt="image" src="https://github.com/user-attachments/assets/d225725a-3567-482f-ab22-ab1aa28ac8dc" />

---

## Task 3: Install a Chart

```bash
helm install my-nginx bitnami/nginx
```

### Release Details

| Field | Value |
|---|---|
| Name | my-nginx |
| Last Deployed | Mon Jul 13 18:46:34 2026 |
| Namespace | default |
| Status | deployed |
| Revision | 1 |
| Chart Version | nginx-25.0.13 |
| App Version | 1.31.2 |

### What Was Created

```bash
kubectl get all
```

**Resources Created:**

- Pod: `my-nginx-785ff86c76-s2n6n` (1/1 Running)
- Service: `my-nginx` (LoadBalancer type, ports 80:30566, 443:30541)
- Deployment: `my-nginx` (1 replica, 1 available)
- ReplicaSet: `my-nginx-785ff86c76` (1 desired, 1 current)
- ServiceAccount: `my-nginx`
- Secret: `my-nginx-tls` (TLS certificate)
- NetworkPolicy: `my-nginx`
- PodDisruptionBudget: `my-nginx`

**Number of Pods Running:** 1
**Service Type Created:** LoadBalancer
**DNS Name:** `my-nginx.default.svc.cluster.local` (port 80)

### Warnings Observed

- **Rolling Tag Warning:** Using `latest` tags in production is not recommended
- **Resource Warning:** No resource limits set for production workloads


> 📸 **Screenshot: `kubectl get all` output**

<img width="1704" height="512" alt="image" src="https://github.com/user-attachments/assets/f04531de-75f7-45dc-9480-9297e9f24067" />


---

## Task 4: Customize with Values

### Using `--set` Overrides

```bash
helm install custom-nginx bitnami/nginx \
  --set replicaCount=3 \
  --set service.type=NodePort
```

**Results:**

| Field | Value |
|---|---|
| Release | custom-nginx |
| Replicas | 3 (vs default 1) |
| Service Type | NodePort (vs LoadBalancer) |
| Pods Created | 3 Running pods |

### Using a Values File

`custom-values.yaml`:

```yaml
replicaCount: 3
service:
  type: NodePort
  port: 80
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
```

**Installation:**

```bash
helm install values-nginx bitnami/nginx -f custom-values.yaml
```

**Verified Values:**

```bash
helm get values values-nginx
```

Output shows all user-supplied values are correctly applied:

- `replicaCount: 3`
- `service:` type: NodePort, port: 80
- `resources:` limits and requests set correctly

### Key Customization Methods

| Method | Use Case | Example |
|---|---|---|
| `--set` | Quick overrides, single values | `--set replicaCount=3` |
| `-f values.yaml` | Complex configurations, multiple values | `-f custom-values.yaml` |
| Nested values | Deep configuration | `--set service.type=NodePort` |


> 📸 **Screenshot: `helm get values values-nginx` output**

<img width="1322" height="504" alt="image" src="https://github.com/user-attachments/assets/186a7b27-e18d-4939-bb39-6555eef224c8" />


---

## Task 5: Upgrade and Rollback

### Upgrade Process

```bash
helm upgrade my-nginx bitnami/nginx --set replicaCount=5
```

**Upgrade Details:**

| Field | Value |
|---|---|
| Release | my-nginx |
| New Revision | 2 |
| Status | deployed |
| Replicas | increased from 1 to 5 |

### Release History

```bash
helm history my-nginx
```

- Revision 1: Install complete (18:46:34)
- Revision 2: Upgrade complete (18:53:42)

### Rollback Process

```bash
helm rollback my-nginx 1
```

**Rollback Results:**

- Rollback was successful
- New revision 3 created
- Revision 3 is the rollback to revision 1

> **Note:** Rollback creates a new revision, doesn't overwrite existing ones.

**Updated History:**

- Revision 1: Install complete (superseded)
- Revision 2: Upgrade complete (superseded)
- Revision 3: Rollback to 1 (deployed)

**Total Revisions After Rollback:** 3

### Comparison with Kubernetes Deployments

| Feature | Helm Rollback | Deployment Rollout |
|---|---|---|
| Scope | Full stack (all resources) | Single Deployment |
| Granularity | Chart-level | Pod-level |
| Revisions | Entire application state | Container image/configuration |
| Complexity | Manages multiple resources | Manages pods only |


> 📸 **Screenshot: `helm history my-nginx`and `helm rollback my-nginx 1` output**

<img width="1734" height="514" alt="image" src="https://github.com/user-attachments/assets/7485afac-5054-4693-a3eb-6d404567f790" />


---

## Task 6: Create Your Own Chart

### Chart Structure

```text
my-app/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default configuration
├── templates/          # Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   ├── tests/
│   │   └── test-connection.yaml
│   ├── _helpers.tpl
│   └── ...
└── charts/             # Dependencies
```

### Chart.yaml Analysis

```yaml
apiVersion: v2
name: my-app
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.16.0"
```

### Template Rendering Examples

**Service Template:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
```

**Deployment Template:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-app.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "my-app.selectorLabels" . | nindent 6 }}
  template:
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
```

### Go Template Syntax

**Basic Variables:**

```go
{{ .Values.replicaCount }}     // Access values
{{ .Chart.Name }}              // Chart metadata
{{ .Release.Name }}            // Release name
{{ .Release.Namespace }}       // Release namespace
```

**Helpers (from `_helpers.tpl`):**

```go
{{- define "my-app.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}
```

**Control Structures:**

```go
{{- if .Values.service.enabled }}    // Conditional
{{- range .Values.env }}              // Loop
{{- with .Values.resources }}         // With block
```

### Custom Values Configuration

Complete `my-app/values.yaml`:

```yaml
replicaCount: 3
image:
  repository: nginx
  tag: "1.25"
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
  port: 80
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
serviceAccount:
  create: true
  automountServiceAccountToken: false
```

### Validation and Preview

```bash
# Lint - Validate chart structure
helm lint my-app
# Output: 1 chart(s) linted, 0 chart(s) failed

# Template - Preview rendered manifests
helm template my-release ./my-app

# Install
helm install my-release ./my-app
```

### Verification Results

**After Installation (replicaCount=3):**

```text
my-release-my-app-5f556f9855-4twpx   1/1 Running
my-release-my-app-5f556f9855-jv862   1/1 Running
my-release-my-app-5f556f9855-z9j6w   1/1 Running
```

**Pods Running:** 3 ✓

**After Upgrade (`--set replicaCount=5`):**

```text
my-release-my-app-5f556f9855-4twpx   1/1 Running
my-release-my-app-5f556f9855-ffllv   1/1 Running  (new)
my-release-my-app-5f556f9855-jv862   1/1 Running
my-release-my-app-5f556f9855-wr6hm   1/1 Running  (new)
my-release-my-app-5f556f9855-z9j6w   1/1 Running
```

**Pods Running After Upgrade:** 5 ✓

> 📸 **Screenshot: Chart folder structure (`tree my-app` / file explorer)**

<img width="577" height="339" alt="image" src="https://github.com/user-attachments/assets/73ac6e55-3fe9-4cc0-b0fc-827d23678c5b" />


> 📸 **Screenshot: `helm lint my-app` output**

<img width="578" height="140" alt="image" src="https://github.com/user-attachments/assets/05800d3b-5aa4-4440-8fe2-228a71081849" />


> 📸 **Screenshot: `kubectl get pods` after install (3 pods)**

<img width="1088" height="478" alt="image" src="https://github.com/user-attachments/assets/17a2ef8d-07df-4c37-a4f6-0b2a8bbff766" />


> 📸 **Screenshot: `kubectl get pods` after upgrade (5 pods)**

<img width="1808" height="1114" alt="image" src="https://github.com/user-attachments/assets/ff3172cb-ba16-433f-a774-2036e32ff6e6" />


---

## Task 7: Clean Up

### Uninstall Commands

```bash
# List releases
helm list

# Uninstall each release
helm uninstall my-nginx
helm uninstall custom-nginx
helm uninstall values-nginx
helm uninstall my-release

# With history retention
helm uninstall my-release --keep-history
```

### Verification

```bash
helm list
# No releases found
```

### Release History vs Complete Removal

| Command | Release Removed | History Kept |
|---|---|---|
| `helm uninstall` | ✅ Yes | ❌ No |
| `helm uninstall --keep-history` | ✅ Yes | ✅ Yes |


---

## Important Helm Commands Summary

| Command | Description |
|---|---|
| `helm version` | Show Helm version |
| `helm env` | Show Helm environment variables |
| `helm repo add` | Add a chart repository |
| `helm repo update` | Update repository cache |
| `helm search repo` | Search for charts |
| `helm show values` | Show default values |
| `helm install` | Install a chart |
| `helm upgrade` | Upgrade a release |
| `helm rollback` | Rollback to previous revision |
| `helm history` | Show release history |
| `helm list` | List all releases |
| `helm status` | Show release status |
| `helm get values` | Show override values |
| `helm get manifest` | Show generated manifests |
| `helm template` | Render templates locally |
| `helm lint` | Validate chart |
| `helm create` | Create new chart |
| `helm uninstall` | Uninstall release |

---

## Key Learnings

- Helm simplifies Kubernetes deployment by packaging multiple resources into a single chart
- Releases provide versioned, trackable deployments of your applications
- Values allow parameterization for different environments (dev/staging/prod)
- Upgrade/Rollback provide safe deployment strategies at the application level
- Custom charts enable reusable, shareable, and versioned application deployments
- Go templates provide powerful dynamic manifest generation
- Multiple releases of the same chart can coexist in the same cluster
