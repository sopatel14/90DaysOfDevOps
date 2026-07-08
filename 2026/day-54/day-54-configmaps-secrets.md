# Day 54 – Kubernetes ConfigMaps & Secrets

Today I learned how Kubernetes **ConfigMaps** and **Secrets** help separate configuration from container images.

Instead of hardcoding configuration values or sensitive credentials inside an application image, Kubernetes allows us to inject them dynamically into Pods using environment variables or mounted files.

---

# Why ConfigMaps and Secrets?

Applications often require configuration such as:

- Environment names
- Port numbers
- Feature flags
- Database usernames
- Passwords
- API keys
- Certificates

Hardcoding these values inside Docker images makes deployments difficult and insecure.

Kubernetes solves this using:

- **ConfigMaps** → Store non-sensitive configuration.
- **Secrets** → Store sensitive information.

---

# ConfigMap vs Secret

| Feature | ConfigMap | Secret |
|----------|-----------|--------|
| Purpose | Non-sensitive configuration | Sensitive information |
| Storage | Plain text | Base64 encoded |
| Encryption | No | Optional (Encryption at Rest) |
| Mount Type | File or Environment Variable | File (tmpfs) or Environment Variable |
| Typical Use | URLs, Ports, Feature Flags | Passwords, API Keys, Tokens |
| Size Limit | 1 MiB | 1 MiB |

---

# Task 1 – Create a ConfigMap from Literals

Created a ConfigMap using command-line literals.

## Command

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=APP_DEBUG=false \
  --from-literal=APP_PORT=8080
```

---

## Verification

```bash
kubectl describe configmap app-config

kubectl get configmap app-config -o yaml
```

Expected Output

```yaml
apiVersion: v1
data:
  APP_DEBUG: "false"
  APP_ENV: production
  APP_PORT: "8080"
kind: ConfigMap
metadata:
  name: app-config
```

### Key Learning

- ConfigMaps store values in plain text.
- No encoding or encryption is applied.
- Easy to inspect and modify.

---

## 📸 Screenshot

<img width="805" height="446" alt="image" src="https://github.com/user-attachments/assets/7f56df7f-1ac5-4067-b3d6-83d49c5bd6da" />

<img width="824" height="261" alt="image" src="https://github.com/user-attachments/assets/f110db96-a0ea-4a8c-983a-d5590a65e389" />



---

# Task 2 – Create a ConfigMap from a File

Created an Nginx configuration file. - vim default.conf 

```bash
server {
    listen 80;
    server_name localhost;

    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
    }

    location /health {
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
```

Created the ConfigMap.

```bash
kubectl create configmap nginx-config \
--from-file=default.conf=default.conf
```

---

## Verification

```bash
kubectl get configmap nginx-config -o yaml
```

### Key Learning

- Entire files can be stored inside ConfigMaps.
- The key becomes the filename when mounted inside a Pod.

---

## 📸 Screenshot

<img width="2090" height="956" alt="image" src="https://github.com/user-attachments/assets/6252475d-db1f-429f-a345-57f7c98d0c53" />

---

# Task 3 – Using ConfigMaps in Pods

## Method 1 – Environment Variables

Created a BusyBox Pod using ConfigMap values. - busybox-env-pod.yaml

```yaml
apiVersion: v1
data:
  APP_DEBUG: "false"
  APP_ENV: production
  APP_PORT: "8080"
kind: ConfigMap
metadata:
  creationTimestamp: "2026-07-08T02:56:35Z"
  name: app-config
  namespace: default
  resourceVersion: "21771"
  uid: 5df64b9f-79e4-4d37-ab0e-b22a9dcb1a6e
souravpatel@Souravs-MacBook-Pro kuberenetes-practice % cat busybox-env-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-env-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "echo 'APP_ENV='$APP_ENV; echo 'APP_DEBUG='$APP_DEBUG; echo 'APP_PORT='$APP_PORT; sleep 3600"]
    envFrom:
    - configMapRef:
        name: app-config
  restartPolicy: Never

```

Applied:

```bash
kubectl apply -f busybox-env-pod.yaml
```

Verification

```bash
kubectl exec busybox-env-pod -- env | grep APP_
```

Output

```
APP_ENV=production
APP_DEBUG=false
APP_PORT=8080
```

---

## 📸 Screenshot

<img width="2050" height="366" alt="image" src="https://github.com/user-attachments/assets/d52ddaad-2935-4cf2-8f96-585772a468e9" />


---

## Method 2 – Volume Mount

Created an Nginx Pod mounting the ConfigMap. - nginx-volume-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-volume-pod
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    volumeMounts:
    - name: nginx-config-volume
      mountPath: /etc/nginx/conf.d
      readOnly: true
    # Add this to reload nginx after config is mounted
    command:
    - /bin/sh
    - -c
    - |
      nginx -g "daemon off;"
  volumes:
  - name: nginx-config-volume
    configMap:
      name: nginx-config
  restartPolicy: Never
```

Applied

```bash
kubectl apply -f nginx-volume-pod.yaml
```

Testing

```bash
kubectl exec nginx-volume-pod -- curl -s http://localhost/health
```

Output

```
healthy
```

### Key Learning

Environment Variables

- Best for small key-value configuration.

Volume Mounts

- Best for full configuration files.

---

## 📸 Screenshot

<img width="2332" height="346" alt="image" src="https://github.com/user-attachments/assets/918f7642-5123-4a4c-bd9c-7283b42348e4" />


---

# Task 4 – Create a Secret

Created Secret from literals.

```bash
kubectl create secret generic db-credentials \
--from-literal=DB_USER=admin \
--from-literal=DB_PASSWORD=s3cureP@ssw0rd
```

Verification

```bash
kubectl get secret db-credentials -o yaml
```

Output

```yaml
data:
  DB_USER: YWRtaW4=
  DB_PASSWORD: czNjdXJlUEBzc3cwcmQ=
```

Decoded values.

```bash
kubectl get secret db-credentials \
-o jsonpath='{.data.DB_USER}' | base64 --decode

kubectl get secret db-credentials \
-o jsonpath='{.data.DB_PASSWORD}' | base64 --decode
```

Output

```
admin
s3cureP@ssw0rd
```

### Key Learning

- Base64 is **encoding**, not encryption.
- Anyone with access can decode it.
- Security comes from RBAC and encryption at rest.

---

## 📸 Screenshot

**Insert Screenshot Here**

<img width="2090" height="500" alt="image" src="https://github.com/user-attachments/assets/ef3452a1-a7f8-4787-8dcb-f5905971c4dd" />

<img width="2328" height="282" alt="image" src="https://github.com/user-attachments/assets/a25187e1-18fd-43ac-8130-7b07228afa39" />

---

# Task 5 – Use Secret in a Pod

Created a BusyBox Pod using Secret as: - # secret-pod.yaml

- Environment Variable
- Mounted Volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: DB_USER
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/db-credentials
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: db-credentials
  restartPolicy: Never

```

Verification

```bash
kubectl exec secret-pod -- env | grep DB_USER

kubectl exec secret-pod -- cat /etc/db-credentials/DB_USER

kubectl exec secret-pod -- cat /etc/db-credentials/DB_PASSWORD

kubectl exec secret-pod -- ls -la /etc/db-credentials/
```

Output

```
DB_USER=admin

admin

s3cureP@ssw0rd
```

### Key Learning

- Secret files contain plaintext values.
- Files are mounted read-only.
- Stored in tmpfs (memory).
- Each Secret key becomes its own file.

---

## 📸 Screenshot

<img width="1750" height="308" alt="image" src="https://github.com/user-attachments/assets/b8af76aa-2150-40e6-a728-01018d9f3fb2" />


---

# Task 6 – Update ConfigMap

Created ConfigMap.

```bash
kubectl create configmap live-config \
--from-literal=message=hello
```

Created monitoring Pod. - # live-config-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: live-config-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command:
    - /bin/sh
    - -c
    - |
      echo "Starting monitoring..."
      while true; do
        echo "$(date): $(cat /etc/config/message)"
        sleep 5
      done
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
      readOnly: true
  volumes:
  - name: config-volume
    configMap:
      name: live-config
  restartPolicy: Never

```

Applied

```bash
kubectl apply -f live-config-pod.yaml
```

Terminal 1

```bash
kubectl logs -f live-config-pod
```

Terminal 2

```bash
kubectl patch configmap live-config \
--type merge \
-p '{"data":{"message":"world"}}'
```

Verification

```bash
kubectl exec live-config-pod -- cat /etc/config/message
```

Output

```
world
```

### Key Learning

- Mounted ConfigMaps update automatically.
- Environment Variables do not update.
- Changes usually propagate within 30–60 seconds.

---

## 📸 Screenshot

<img width="1734" height="1024" alt="image" src="https://github.com/user-attachments/assets/d5f8b6fc-a08d-450a-8498-f51ea2a6dc3a" />

---

# ConfigMap Update Behavior

| Injection Method | Auto Updates | When |
|------------------|--------------|------|
| Environment Variable | ❌ No | Pod Restart Required |
| Volume Mount | ✅ Yes | Approximately 60 Seconds |

---

# Why Base64 is Encoding, Not Encryption

Encoding

```bash
echo -n "secret" | base64

echo "c2VjcmV0" | base64 --decode
```

Encryption

Requires a secret key for decryption.

Base64 simply converts binary data into text.

It does **not** provide security.

---

# Best Practices

- Store sensitive data in Secrets.
- Store application configuration in ConfigMaps.
- Use Environment Variables for small settings.
- Use Volume Mounts for configuration files.
- Enable Encryption at Rest in production clusters.
- Restrict Secret access using RBAC.
- Never log Secret values.
- Review YAML using `--dry-run=client -o yaml`.

---

# Useful Commands

```bash
kubectl get configmaps

kubectl describe configmap app-config

kubectl get configmap app-config -o yaml

kubectl get secrets

kubectl describe secret db-credentials

kubectl get secret db-credentials -o yaml

kubectl get endpoints

kubectl logs -f live-config-pod
```

---

# Cleanup

Delete Pods

```bash
kubectl delete pod busybox-env-pod
kubectl delete pod nginx-volume-pod
kubectl delete pod secret-pod
kubectl delete pod live-config-pod
```

Delete ConfigMaps

```bash
kubectl delete configmap app-config
kubectl delete configmap nginx-config
kubectl delete configmap live-config
```

Delete Secret

```bash
kubectl delete secret db-credentials
```

Verify

```bash
kubectl get pods

kubectl get configmaps

kubectl get secrets
```

---

# Files Created

```
day-54-configmaps-secrets.md

busybox-env-pod.yaml

nginx-volume-pod.yaml

secret-pod.yaml

live-config-pod.yaml
```

---

# Summary

Today I learned:

- How ConfigMaps separate configuration from container images.
- How to create ConfigMaps from literals and files.
- How to inject ConfigMaps as environment variables and mounted volumes.
- How to create Kubernetes Secrets.
- Why Secrets use Base64 encoding.
- The difference between encoding and encryption.
- How Secrets are consumed inside Pods.
- How ConfigMap updates automatically propagate to mounted volumes.
- Best practices for securely managing configuration in Kubernetes.

### Key Takeaway

ConfigMaps and Secrets allow applications to remain portable and environment-independent by separating configuration from application code. ConfigMaps manage non-sensitive settings, while Secrets securely handle credentials. Volume-mounted ConfigMaps automatically receive updates without restarting Pods, making them ideal for dynamic configuration.
