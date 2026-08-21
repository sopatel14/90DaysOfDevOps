# Day 79: Creating a Custom Helm Chart for AI-BankApp

## 📋 Overview

Yesterday I deployed MySQL using a pre-built community chart (Bitnami). Today I went a level deeper: converting the AI-BankApp's own **12 raw Kubernetes manifests** into a single, custom, reusable **Helm chart**. The AI-BankApp stack has three moving parts — a Spring Boot banking app, a MySQL database, and an Ollama AI chatbot — and by the end of today all three deploy together with one command.

**Repository:** [AI-BankApp-DevOps](https://github.com/TrainWithShubham/AI-BankApp-DevOps) (branch: `feat/gitops`)

### What I Built

- ✅ A custom Helm chart that deploys the entire AI-BankApp stack
- ✅ Templates for Deployments, Services, ConfigMap, Secrets, PVCs, and HPA
- ✅ Init containers and lifecycle hooks preserved from the original manifests
- ✅ Chart validated with `helm lint` and `helm template`

---

## 1.From Manifests to a Chart

Building a custom chart is really just a **mechanical transformation**: every hardcoded value in a raw manifest becomes a variable pulled from `values.yaml`, injected through Go's templating syntax (`{{ }}`). The manifest's *shape* stays the same — it's still a Deployment, a Service, a Secret — but now:

- The same template can render differently per environment (dev/staging/prod) just by swapping the values file.
- Optional components (like Ollama) can be switched on/off with a single boolean, instead of deleting/adding files.
- Secrets are auto-encoded instead of manually base64'd.
- The whole stack becomes one versioned, installable, rollback-able unit instead of 12 independent files with no shared lifecycle.

This is the core value proposition of Helm charts beyond just using someone else's (like Bitnami's yesterday): **your own application** becomes as easy to deploy, configure, and roll back as any community chart.

---

## 2. Original Manifests Analysis

The raw `k8s/` directory contained 12 YAML files, each responsible for one piece of the stack:

| File | Purpose |
|---|---|
| `namespace.yml` | Creates the `bankapp` namespace |
| `configmap.yml` | MySQL host, port, database, Ollama URL |
| `secrets.yml` | MySQL credentials (base64 encoded) |
| `pv.yml` | StorageClass (gp3 via EBS CSI) |
| `pvc.yml` | PVCs for MySQL (5Gi) and Ollama (10Gi) |
| `bankapp-deployment.yml` | BankApp with init containers, probes, `envFrom` |
| `mysql-deployment.yml` | MySQL with EBS volume mount, probes |
| `ollama-deployment.yml` | Ollama with `postStart` model pull, probes |
| `service.yml` | ClusterIP services for all 3 components |
| `hpa.yml` | HPA for BankApp (2–4 replicas, 70% CPU) |
| `gateway.yml` | Envoy Gateway + HTTPRoute + TLS |
| `cert-manager.yml` | Let's Encrypt `ClusterIssuer` |

Mapping every file to its purpose *before* templating is what made the conversion straightforward — each raw file became (roughly) one template file.

---

## 3. Chart Structure

```
bankapp/
├── Chart.yaml              # Chart metadata
├── values.yaml              # All configurable parameters
├── templates/
│   ├── _helpers.tpl          # Helper functions
│   ├── NOTES.txt              # Post-installation notes
│   ├── bankapp-deployment.yaml
│   ├── mysql-deployment.yaml
│   ├── ollama-deployment.yaml
│   ├── configmap.yaml
│   ├── secrets.yaml
│   ├── services.yaml
│   ├── hpa.yaml
│   └── storage.yaml
```

### Scaffolding the Chart

```bash
cd AI-BankApp-DevOps
mkdir helm-chart && cd helm-chart
helm create bankapp
rm -rf bankapp/templates/*.yaml bankapp/templates/tests/
```

`helm create` scaffolds a default chart with example templates (a sample Deployment, Service, Ingress, etc.). Since I'm writing my own templates from the raw manifests, I deleted the generated `.yaml` files and the `tests/` folder — but **kept `_helpers.tpl` and `NOTES.txt`**, since those get customized rather than replaced.

---

## 4. Chart.yaml

```yaml
apiVersion: v2
name: bankapp
description: AI-BankApp -- Spring Boot banking application with MySQL and Ollama AI chatbot
type: application
version: 0.1.0
appVersion: "1.0.0"
maintainers:
  - name: TrainWithShubham
    url: https://github.com/TrainWithShubham
keywords:
  - bankapp
  - spring-boot
  - mysql
  - ollama
  - ai
```

Same distinction as yesterday: `version` (0.1.0) is the chart's own version, `appVersion` (1.0.0) tracks the BankApp application itself. They evolve independently.

---

## 5. values.yaml — Complete File and Explanation

Every hardcoded value from the 12 raw manifests was extracted here so nothing lives directly inside a template.

```yaml
# BankApp configuration
bankapp:
  replicaCount: 4
  image:
    repository: trainwithshubham/ai-bankapp-eks
    tag: "latest"
    pullPolicy: Always
  resources:
    requests:
      memory: "256Mi"
      cpu: "250m"
    limits:
      memory: "512Mi"
      cpu: "500m"
  service:
    type: ClusterIP
    port: 8080
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 4
    targetCPUUtilization: 70

# MySQL configuration
mysql:
  enabled: true
  image:
    repository: mysql
    tag: "8.0"
  resources:
    requests:
      memory: "256Mi"
      cpu: "250m"
    limits:
      memory: "512Mi"
      cpu: "500m"
  persistence:
    size: 5Gi
    storageClass: gp3

# Ollama AI configuration
ollama:
  enabled: true
  image:
    repository: ollama/ollama
    tag: "latest"
  model: tinyllama
  resources:
    requests:
      memory: "2Gi"
      cpu: "900m"
    limits:
      memory: "2.5Gi"
      cpu: "1500m"
  persistence:
    size: 10Gi
    storageClass: gp3

# Shared configuration
config:
  mysqlDatabase: bankappdb
  ollamaUrl: ""  # Auto-generated from service name if empty

# Secrets
secrets:
  mysqlRootPassword: Test@123
  mysqlUser: root
  mysqlPassword: Test@123

# Storage
storageClass:
  create: true
  name: gp3
  provisioner: ebs.csi.aws.com

# Gateway (optional -- for EKS with Envoy Gateway)
gateway:
  enabled: false
  hostname: ""
  tls:
    enabled: false
```

### Field-by-Field Explanation

- **`bankapp.*`** — Everything about the Spring Boot app: replica count, image, resource limits, service type/port, and HPA settings. `autoscaling.enabled: true` means the HPA (not `replicaCount`) controls the running pod count.
- **`mysql.*`** — MySQL image/tag, resource requests/limits, and PVC size/storage class. `enabled: true` gates the entire MySQL Deployment, PVC, and Service — flipping it to `false` removes them all.
- **`ollama.*`** — Same pattern as MySQL, plus `model: tinyllama`, which controls which LLM gets pulled at startup. Ollama needs far more memory/CPU headroom (2Gi/900m requests) since it's running an actual model.
- **`config.*`** — Shared, non-secret configuration injected into the ConfigMap. `ollamaUrl` is left blank so the template can auto-derive it from the release name if not explicitly overridden.
- **`secrets.*`** — Plaintext values here get **auto base64-encoded** by the Secret template using Helm's `b64enc` function — no manual encoding required.
- **`storageClass.*`** — Controls whether the chart creates its own `StorageClass` (needed on EKS with the EBS CSI driver) or relies on one that already exists (like Kind's default `standard` class).
- **`gateway.*`** — Optional Envoy Gateway + TLS config for EKS ingress, disabled by default since it's not needed for local Kind testing.

**Compare:** the raw `k8s/secrets.yml` had base64-encoded credentials hardcoded directly in the file. The Helm chart instead stores plaintext in `values.yaml` and encodes it at render time — each environment can override credentials without ever touching YAML syntax.

---

## 6. Raw Manifests vs Helm Templates — Side-by-Side

### ConfigMap

**Raw Manifest (hardcoded):**
```yaml
data:
  MYSQL_HOST: "mysql-service"
  MYSQL_PORT: "3306"
  MYSQL_DATABASE: "bankappdb"
  OLLAMA_URL: "http://ollama-service:11434"
```

**Helm Template (dynamic):**
```yaml
data:
  MYSQL_HOST: {{ include "bankapp.fullname" . }}-mysql
  MYSQL_PORT: "3306"
  MYSQL_DATABASE: {{ .Values.config.mysqlDatabase | quote }}
  OLLAMA_URL: {{ default (printf "http://%s-ollama:11434" (include "bankapp.fullname" .)) .Values.config.ollamaUrl | quote }}
```

The host names are no longer hardcoded strings — they're derived from the release name via `bankapp.fullname`, so multiple releases of the same chart never collide.

### Secret

**Raw Manifest (manually base64 encoded):**
```yaml
data:
  MYSQL_ROOT_PASSWORD: "VGVzdEAxMjM="
  MYSQL_USER: "cm9vdA=="
  MYSQL_PASSWORD: "VGVzdEAxMjM="
```

**Helm Template (auto-encoded):**
```yaml
data:
  MYSQL_ROOT_PASSWORD: {{ .Values.secrets.mysqlRootPassword | b64enc | quote }}
  MYSQL_USER: {{ .Values.secrets.mysqlUser | b64enc | quote }}
  MYSQL_PASSWORD: {{ .Values.secrets.mysqlPassword | b64enc | quote }}
```

No more running credentials through `base64` by hand and pasting the output — `b64enc` does it at render time straight from `values.yaml`.

### Init Containers (Conditional)

```yaml
initContainers:
  - name: wait-for-mysql
    image: busybox:1.36
    command: ["/bin/sh", "-c", "until nc -z {{ include "bankapp.fullname" . }}-mysql 3306; do sleep 2; done"]
  {{- if .Values.ollama.enabled }}
  - name: wait-for-ollama
    image: busybox:1.36
    command: ["/bin/sh", "-c", "until nc -z {{ include "bankapp.fullname" . }}-ollama 11434; do sleep 2; done"]
  {{- end }}
```

The `wait-for-ollama` init container only renders **if Ollama is enabled** — this is the same conditional pattern used throughout the chart to make components optional.

---

## 7. Writing the Core Templates

### storage.yaml (from `pv.yml` + `pvc.yml`)

Conditionally creates a `StorageClass` and PVCs for MySQL and Ollama, each gated by its own `enabled` flag:

```yaml
{{- if .Values.storageClass.create }}
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: {{ .Values.storageClass.name }}
provisioner: {{ .Values.storageClass.provisioner }}
parameters:
  type: gp3
  fsType: ext4
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
{{- end }}
---
{{- if .Values.mysql.enabled }}
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ include "bankapp.fullname" . }}-mysql-pvc
  namespace: {{ .Release.Namespace }}
spec:
  storageClassName: {{ .Values.mysql.persistence.storageClass }}
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: {{ .Values.mysql.persistence.size }}
{{- end }}
```

*(Ollama's PVC mirrors this exact pattern.)*

### bankapp-deployment.yaml (from `bankapp-deployment.yml`)

Key template decisions:

- Init containers dynamically reference MySQL/Ollama service names via `{{ include "bankapp.fullname" . }}` instead of hardcoded service names.
- The `wait-for-ollama` init container is **conditional**, matching whether Ollama is enabled.
- Health probes use `/actuator/health` — Spring Boot's built-in health endpoint.
- `replicas` is **omitted entirely** when autoscaling is enabled, so the HPA (not the Deployment spec) owns the replica count — avoiding the classic "Helm and HPA fighting over replica count" problem.

### mysql-deployment.yaml / ollama-deployment.yaml

Both follow the same shape: wrapped in `{{- if .Values.<component>.enabled }}` so the entire Deployment simply doesn't render if the component is turned off. MySQL uses `mysqladmin ping` for its probes; Ollama uses a `postStart` lifecycle hook to pull the configured model (`{{ .Values.ollama.model }}`) once the server is up — meaning switching LLMs is now a one-line values change instead of editing a shell command inside YAML.

---

## 8. Services and HPA

`services.yaml` defines three `ClusterIP` services (MySQL, Ollama — conditional, and the main BankApp service), while `hpa.yaml` is entirely wrapped in `{{- if .Values.bankapp.autoscaling.enabled }}` and reads `minReplicas`, `maxReplicas`, and `targetCPUUtilization` straight from values — including scale-up/scale-down behavior windows (30s up, 300s down) to avoid flapping.

---

## 9. Go Template Syntax Cheat Sheet

| Function | Purpose | Example |
|---|---|---|
| `{{ .Values.key }}` | Access a value from `values.yaml` | `{{ .Values.bankapp.replicaCount }}` |
| `{{ include "name" . }}` | Call a named template (pipeable) | `{{ include "bankapp.fullname" . }}` |
| `{{- if .Values.enabled }}` | Conditional, with whitespace trimming | `{{- if .Values.ollama.enabled }}` |
| `{{ .Value \| default "fallback" }}` | Set a default if the value is empty | `{{ default 8080 .Values.port }}` |
| `{{ .Value \| b64enc }}` | Base64-encode a string | `{{ .Values.password \| b64enc }}` |
| `{{ .Values.resources \| toYaml \| nindent 4 }}` | Convert an object to YAML with proper indentation | Resources block |
| `{{ printf "http://%s:11434" .Values.host }}` | String formatting | Service URL construction |
| `{{ .Release.Namespace }}` | Current release's namespace | `namespace: {{ .Release.Namespace }}` |

Notes worth remembering:
- `{{-` trims leading whitespace, `-}}` trims trailing whitespace — needed to keep rendered YAML clean when using `if`/`end` blocks.
- `include` returns a string and can be piped (`| nindent 4`); `template` streams output directly and **cannot** be piped.
- `toYaml` + `nindent` is the standard pattern for dropping an entire values object (like `resources`) into a template as valid YAML.

---

## 10. Validation and Deployment

### Step 1: Lint the Chart

```bash
helm lint bankapp/
```

```
==> Linting bankapp/
[INFO] Chart.yaml: icon is recommended
1 chart(s) linted, 0 chart(s) failed
```

### Step 2: Render Templates Locally

```bash
helm template my-bankapp bankapp/
```

Every `{{ }}` in the output should resolve to a real value — this is the best debugging tool before ever touching a real cluster.


### Step 3: Render with Overrides

```bash
helm template my-bankapp bankapp/ \
  --set bankapp.image.tag=abc1234 \
  --set bankapp.replicaCount=2 \
  --set ollama.enabled=false
```

### Step 4: Dry Run Against the Cluster

```bash
helm install my-bankapp bankapp/ --dry-run --debug -n bankapp --create-namespace
```

### Step 5: Deploy for Real (on Kind)

Kind uses its own default `standard` StorageClass, so the chart's own `StorageClass` creation is skipped:

```bash
helm install my-bankapp bankapp/ \
  -n bankapp --create-namespace \
  --set storageClass.create=false \
  --set mysql.persistence.storageClass=standard \
  --set ollama.persistence.storageClass=standard
```

### Step 6: Verify

```bash
helm list -n bankapp
kubectl get all -n bankapp
kubectl get pvc -n bankapp
kubectl get configmap,secret -n bankapp
kubectl get pods -n bankapp -w
```

Expected pods once everything settles (Ollama takes longer since it pulls a model on startup):

```
NAME                                 READY   STATUS    RESTARTS   AGE
my-bankapp-7567587896-w8jw5          1/1     Running   0          4m55s
my-bankapp-mysql-6788576b64-npphw    1/1     Running   0          31m
my-bankapp-ollama-55bb44df54-wm59j   1/1     Running   0          31m
```

<img width="780" height="106" alt="image" src="https://github.com/user-attachments/assets/81ab2c50-d7d1-4e9a-9984-927315bab63e" />


### Step 7: Access the App

```bash
kubectl port-forward svc/my-bankapp-bankapp-service -n bankapp 8080:8080
```

Open `http://localhost:8080` — the AI-BankApp login page.

<img width="1671" height="820" alt="image" src="https://github.com/user-attachments/assets/3592945f-07d4-4ebc-9a52-a4d7b1dd30fa" />

<img width="1669" height="876" alt="Screenshot 2026-08-21 at 9 16 06 AM" src="https://github.com/user-attachments/assets/c0bc0b93-de14-4a9e-aed9-90806c1ca6f1" />


---

## 11. Database Setup

The app expects an `accounts` table with test users, so these were inserted manually post-deploy:

```bash
kubectl exec -it my-bankapp-mysql-6788576b64-npphw -n bankapp -- mysql -uroot -pTest@123 -e "
USE bankappdb;
INSERT INTO accounts (balance, password, username) VALUES
(1000.00, 'Test@123', 'root'),
(5000.00, 'Test@123', 'admin');
"

kubectl exec -it my-bankapp-mysql-6788576b64-npphw -n bankapp -- mysql -uroot -pTest@123 -e "USE bankappdb; SELECT id, username, balance FROM accounts;"
```

**Login Credentials**

| Username | Password |
|---|---|
| root | Test@123 |
| admin | Test@123 |


---

## 12. Testing Component Toggling

One of the biggest wins of templating with conditionals is that entire components become **one boolean away** from being removed — no file deletion required.

### Disable Ollama

```bash
helm template my-bankapp bankapp/ --set ollama.enabled=false
```

This single flag removes, in one shot:
- The Ollama Deployment
- The Ollama Service
- The Ollama PVC
- The `wait-for-ollama` init container inside the BankApp Deployment

### Re-enable Ollama (default)

```bash
helm template my-bankapp bankapp/ --set ollama.enabled=true
```

All Ollama-related resources render again, exactly as before.

---

## 13. Validation Commands Reference

```bash
# Lint the chart
helm lint bankapp/

# Render templates locally
helm template my-bankapp bankapp/

# Dry run against cluster
helm install my-bankapp bankapp/ --dry-run --debug -n bankapp --create-namespace

# Check resources
kubectl get all -n bankapp
kubectl get configmap,secret -n bankapp
kubectl get pvc -n bankapp
kubectl get hpa -n bankapp

# View logs
kubectl logs -f my-bankapp-7567587896-c2jrv -n bankapp
```

---

## 14. Clean Up

```bash
helm uninstall my-bankapp -n bankapp
kubectl delete namespace bankapp
```

---

## 15. Summary

| Aspect | Before (Raw Manifests) | After (Helm Chart) |
|---|---|---|
| **Deployment** | `kubectl apply -f k8s/` (12 commands) | `helm install` (1 command) |
| **Configuration** | Edit YAML files directly | Override with `--set` or values files |
| **Environment management** | Multiple copies of manifests | One chart, different values files |
| **Rollbacks** | Manual re-apply | `helm rollback` with version history |
| **Reusability** | Copy-paste between projects | One chart, versioned and shareable |
| **Secrets** | Manually base64 encoded | Auto-encoded with `b64enc` |

### Key Learning

Helm transforms Kubernetes management from **manual YAML editing** into **parameterized deployments** — the same chart can serve any environment, any credential set, and even switch entire components on or off, all without touching a template file. This is what makes the AI-BankApp's stack genuinely portable across dev, staging, and production, and safely rollback-able if a change ever breaks something in production.
