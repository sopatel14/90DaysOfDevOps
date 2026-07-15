# Day 60 – Capstone: Deploy WordPress + MySQL on Kubernetes

---

## Task 1: Create the Namespace

```bash
# Create the namespace
kubectl create namespace capstone

# Set as default context
kubectl config set-context --current --namespace=capstone

# Verify
kubectl get namespaces
```

**Expected Output:**
```
namespace/capstone created
Context "docker-desktop" modified.
```


**Q1: What is a Namespace and why is it important for this capstone?**

A Namespace is a Kubernetes abstraction that provides virtual clusters within a single physical cluster. It's crucial for this capstone because:
- It isolates the WordPress+MySQL resources from other applications
- Provides resource quota management and access control boundaries
- Makes resource organization and cleanup easier (all resources in one namespace can be deleted together)
- Prevents naming conflicts with other applications

**Q2: How does setting the namespace as default affect kubectl commands?**

- All kubectl commands without `-n` or `--namespace` flag will operate in the `capstone` namespace
- Reduces command verbosity and potential errors from forgetting to specify namespace
- The current context in the kubeconfig file is updated
- Can be reverted with `kubectl config set-context --current --namespace=default`

---

## Task 2: Deploy MySQL

### Create MySQL Secret
**File:** `mysql-secret.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: capstone
type: Opaque
stringData:
  MYSQL_ROOT_PASSWORD: rootpassword123
  MYSQL_DATABASE: wordpress
  MYSQL_USER: wpuser
  MYSQL_PASSWORD: wppassword123
```

Apply:
```bash
kubectl apply -f mysql-secret.yaml
```

### Create MySQL Headless Service
**File:** `mysql-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: capstone
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
```

Apply:
```bash
kubectl apply -f mysql-service.yaml
```

### Create MySQL StatefulSet
**File:** `mysql-statefulset.yaml`

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: capstone
spec:
  serviceName: mysql
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        envFrom:
        - secretRef:
            name: mysql-secret
        ports:
        - containerPort: 3306
        resources:
          requests:
            cpu: 250m
            memory: 512Mi
          limits:
            cpu: 500m
            memory: 1Gi
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

Apply:
```bash
kubectl apply -f mysql-statefulset.yaml
```

### Verify MySQL
```bash
# Check if MySQL pod is running
kubectl get pods

# Verify MySQL is working
kubectl exec -it mysql-0 -- mysql -u wpuser -pwppassword123 -e "SHOW DATABASES;"
```

**Expected Output:**
```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| wordpress          |
+--------------------+
```


**Q1: Why use a StatefulSet instead of a Deployment for MySQL?**

- **Stable network identity:** Each pod gets a predictable DNS name (`mysql-0.mysql.capstone.svc.cluster.local`)
- **Ordered deployment and scaling:** Pods are created and terminated in order (important for replication)
- **Stable persistent storage:** Each pod has its own PVC that persists across rescheduling
- **Stateful applications:** MySQL maintains state (data) that must survive pod restarts
- **Headless service:** Allows direct pod-to-pod communication without load balancing

**Q2: What is a Headless Service and why is it used here?**

A Headless Service (`clusterIP: None`) provides:
- **Direct pod DNS:** Returns pod IPs instead of a cluster IP
- **Discovery:** Enables pods to discover each other via DNS
- **StatefulSet support:** Required for StatefulSets to manage pod identities
- **No load balancing:** Applications handle their own load balancing (MySQL replication)
- **Stable hostnames:** Each pod gets `pod-name.service-name.namespace.svc.cluster.local`

**Q3: What does `volumeClaimTemplates` do in a StatefulSet?**

- **Dynamic provisioning:** Automatically creates PVCs for each pod replica
- **Persistent storage:** Ensures each MySQL pod gets its own storage
- **Stable binding:** Each PVC is bound to a specific pod identity
- **Survives rescheduling:** Storage persists even if pod is recreated
- **Template-based:** Defines storage requirements (size, access modes, storage class)
- **Ordered creation:** PVCs are created before pods, ensuring storage is ready

---

## Task 3: Deploy WordPress

### Create WordPress ConfigMap
**File:** `wordpress-configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: wordpress-config
  namespace: capstone
data:
  WORDPRESS_DB_HOST: mysql-0.mysql.capstone.svc.cluster.local:3306
  WORDPRESS_DB_NAME: wordpress
```

Apply:
```bash
kubectl apply -f wordpress-configmap.yaml
```

### Create WordPress Deployment (Working Version)
**File:** `wordpress-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  namespace: capstone
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress:6.4.3
        ports:
        - containerPort: 80
        env:
        - name: WORDPRESS_DB_HOST
          value: mysql-0.mysql.capstone.svc.cluster.local:3306
        - name: WORDPRESS_DB_NAME
          value: wordpress
        - name: WORDPRESS_DB_USER
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_USER
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_PASSWORD
        resources:
          requests:
            cpu: 100m
            memory: 256Mi
          limits:
            cpu: 200m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /wp-admin/install.php
            port: 80
          initialDelaySeconds: 120
          periodSeconds: 10
          failureThreshold: 5
        readinessProbe:
          httpGet:
            path: /wp-admin/install.php
            port: 80
          initialDelaySeconds: 90
          periodSeconds: 5
          failureThreshold: 3
```

Apply:
```bash
kubectl apply -f wordpress-deployment.yaml
```

### Verify WordPress
```bash
# Check pod status
kubectl get pods -n capstone

# Wait for both pods to be ready
kubectl wait --for=condition=ready pod -l app=wordpress --timeout=180s -n capstone

# Check logs
kubectl logs -l app=wordpress -n capstone --tail=20
```

**Expected Output:**
```
NAME                         READY   STATUS    RESTARTS   AGE
mysql-0                      1/1     Running   0          2m
wordpress-5797855fbc-hnpzd   1/1     Running   0          30s
wordpress-5797855fbc-krtmf   1/1     Running   0          30s
```

**Q1: How does WordPress find the MySQL database?**

- **Environment variables:** `WORDPRESS_DB_HOST` and `WORDPRESS_DB_NAME` from ConfigMap
- **DNS resolution:** The hostname `mysql-0.mysql.capstone.svc.cluster.local` resolves to the MySQL pod IP
- **Service discovery:** Kubernetes DNS automatically resolves service names
- **ClusterDNS:** The `.cluster.local` domain is the internal cluster DNS zone

**Q2: Why separate database credentials into Secret and configuration into ConfigMap?**

- **Secret:** Contains sensitive data (passwords, user credentials)
  - Encoded in base64 (though not encrypted by default)
  - Can be encrypted at rest with encryption providers
  - Access controlled via RBAC
- **ConfigMap:** Contains non-sensitive configuration
  - Plain text, easier to view and modify
  - Can be updated without recreating pods (if using volume mounts)
  - Separates configuration from application code

**Q3: What is the purpose of liveness and readiness probes?**

- **Readiness Probe:**
  - Determines when a pod is ready to accept traffic
  - If it fails, the pod is removed from service endpoints
  - Prevents traffic to starting/unhealthy pods
  - Initial delay: 90s (allows WordPress to start)
- **Liveness Probe:**
  - Determines if the pod needs to be restarted
  - If it fails, Kubernetes restarts the pod
  - Prevents hanging/deadlocked applications
  - Initial delay: 120s (allows WordPress to fully initialize)
- Both check `/wp-admin/install.php` to verify PHP/WordPress is responding

---

## Task 4: Expose WordPress

### Create WordPress NodePort Service
**File:** `wordpress-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  namespace: capstone
spec:
  type: NodePort
  selector:
    app: wordpress
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

Apply:
```bash
kubectl apply -f wordpress-service.yaml
```

### Access WordPress
```bash
# Port forward to access WordPress
kubectl port-forward svc/wordpress 8080:80 -n capstone
```

Then open browser at: `http://localhost:8080`

### Complete WordPress Setup

Fill in the WordPress setup wizard:
- **Site Title:** "My Kubernetes Blog"
- **Username:** "admin"
- **Password:** "SecurePassword123!"
- **Email:** "admin@example.com"

Log in to WordPress admin, then create a test blog post:
- **Title:** "Kubernetes Capstone Day 60"
- **Content:** "Successfully deployed WordPress on Kubernetes with persistent storage and self-healing!"
- Click **Publish**


**Q1: What is a NodePort Service and when should you use it?**

NodePort is a service type that:
- **Exposes service:** On each node's IP at a static port (30000–32767)
- **Accessible externally:** Can be accessed via `<NodeIP>:<NodePort>`
- **Use cases:** Development/testing environments, when a load balancer is not available, direct node access is acceptable, Minikube/Kind local development
- **Limitations:** Limited port range (30000–32767), not recommended for production (use LoadBalancer or Ingress), security considerations (exposes node ports)

**Q2: How can WordPress be accessed without a load balancer?**

- **NodePort:** Direct access via node IP and port (30080)
- **Port-forward:** `kubectl port-forward` for local testing
- **Ingress:** Using an Ingress controller for HTTP/HTTPS routing
- **ClusterIP + kubectl proxy:** Using `kubectl proxy` to access ClusterIP
- **kubectl port-forward:** For individual pod access

---

## Task 5: Test Self-Healing and Persistence

### Test WordPress Self-Healing
```bash
# Get current pods
kubectl get pods -n capstone

# Delete a WordPress pod
kubectl delete pod -l app=wordpress -n capstone

# Watch recreation
kubectl get pods -n capstone -w
```

### Test MySQL Self-Healing
```bash
# Delete MySQL pod
kubectl delete pod mysql-0 -n capstone

# Watch recreation
kubectl get pods -n capstone -w

# Wait for MySQL to be ready
kubectl wait --for=condition=ready pod mysql-0 --timeout=120s -n capstone
```

### Verify Persistence
```bash
# Check if blog post still exists
# Refresh the browser at WordPress site
# Login and check if the blog post is still present
```

**Expected Verification:**
- WordPress pods recreate quickly (< 30 seconds)
- MySQL pod recreates (< 60 seconds)
- Blog post remains intact after both resets
- Site continues to function normally

### Theory Questions

**Q1: Why does deleting a WordPress pod not affect the site?**

- **ReplicaSet:** Ensures the desired number of replicas (2) are running
- **Controller:** Deployment controller detects the missing pod and creates a new one
- **Stateless:** WordPress is stateless (session data might be lost, but content is in MySQL)
- **Service:** Load balances traffic to remaining replicas during replacement
- **Fast recovery:** New pod starts quickly and becomes ready

**Q2: How does MySQL ensure data persistence?**

- **PersistentVolumeClaims:** Each MySQL pod has its own PVC
- **Volume mounts:** Data is stored at `/var/lib/mysql`
- **StatefulSet stability:** Even after deletion, the pod retains its identity
- **PV/PVC binding:** Storage is bound to the pod's identity (`mysql-0`)
- **Volume reclaim policy:** Data persists unless PVC/PV is deleted
- **Ordered startup:** Pod restarts with the same PVC and data

---

## Task 6: Set Up HPA

### Create HPA Manifest
**File:** `wordpress-hpa.yaml`

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: wordpress-hpa
  namespace: capstone
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: wordpress
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

Apply:
```bash
kubectl apply -f wordpress-hpa.yaml
```

### Verify HPA
```bash
# Check HPA status
kubectl get hpa -n capstone

# Detailed HPA status
kubectl describe hpa wordpress-hpa -n capstone
```

**Expected Output:**
```
NAME             REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
wordpress-hpa    Deployment/wordpress    1%/50%    2         10        2          3m24s
```

### Get Complete Picture
```bash
kubectl get all -n capstone
```

**Expected Output:**
```
NAME                             READY   STATUS    RESTARTS   AGE
pod/mysql-0                      1/1     Running   0          5m52s
pod/wordpress-5797855fbc-hnpzd   1/1     Running   0          13m
pod/wordpress-5797855fbc-krtmf   1/1     Running   0          6m36s

NAME                TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
service/mysql       ClusterIP   None          <none>        3306/TCP       78m
service/wordpress   NodePort    10.96.8.210   <none>        80:30080/TCP   13m

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/wordpress   2/2     2            2           13m

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/wordpress-5797855fbc   2         2         2       13m

NAME                     READY   AGE
statefulset.apps/mysql   1/1     78m

NAME                                                REFERENCE              TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/wordpress-hpa   Deployment/wordpress   cpu: 1%/50%   2         10        2          3m24s
```

**Q1: How does the Horizontal Pod Autoscaler work in this scenario?**

- **Metrics collection:** Kubernetes metrics server collects CPU usage
- **Target:** Maintains average CPU utilization at 50%
- **Scaling logic:** Current CPU > 50% adds pods up to max (10); current CPU < 50% removes pods down to min (2)
- **Calculation:** `Desired replicas = ceil(currentReplicas * (currentUtilization / targetUtilization))`
- **Cooldown:** Avoids flapping with stabilization window (default 5 minutes)
- **Pod selection:** New pods are created, old pods are terminated gracefully

**Q2: What happens if you set minReplicas to 2 in the HPA?**

- **Always has 2 replicas:** Even with no traffic/CPU usage
- **High availability:** Prevents single point of failure
- **Zero downtime:** One pod can go down while the other handles traffic
- **Resource overhead:** More resource consumption than min 1
- **Initial scaling:** Pods remain at 2 unless CPU exceeds 50%

---

## Task 7: (Bonus) Compare with Helm

### Install WordPress with Helm
```bash
# Create a separate namespace
kubectl create namespace helm-wp

# Add Bitnami repo
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Install WordPress via Helm
helm install wp-helm bitnami/wordpress -n helm-wp

# Check resources
kubectl get all -n helm-wp
```

**Expected Output:**
```
NAME                                     READY   STATUS     RESTARTS   AGE
pod/wp-helm-mariadb-0                    0/1     Init:0/1   0          9s
pod/wp-helm-wordpress-6774676675-8pq95   0/1     Init:0/1   0          9s

NAME                               TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/wp-helm-mariadb            ClusterIP      10.96.230.108   <none>        3306/TCP                     9s
service/wp-helm-mariadb-headless   ClusterIP      None            <none>        3306/TCP                     9s
service/wp-helm-wordpress          LoadBalancer   10.96.96.65     <pending>     80:31247/TCP,443:30493/TCP   9s
```

### Check PVCs
```bash
kubectl get pvc -n helm-wp
```

**Expected Output:**
```
NAME                     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS
data-wp-helm-mariadb-0   Bound    pvc-acefc711-6927-4e31-bdc6-7a9469669b01   8Gi        RWO            standard
wp-helm-wordpress        Bound    pvc-c0151a76-5596-4fa9-9360-87d6ad817e37   10Gi       RWO            standard
```

### Compare Resource Counts
```bash
# Count resources in manual deployment
echo "Manual Deployment (capstone):"
kubectl get all -n capstone --no-headers | wc -l

# Count resources in Helm deployment
echo "Helm Deployment (helm-wp):"
kubectl get all -n helm-wp --no-headers | wc -l
```

### Clean Up Helm
```bash
# Uninstall Helm release
helm uninstall wp-helm -n helm-wp

# Delete namespace
kubectl delete namespace helm-wp
```

**Q1: How many resources did each approach create? Which gives more control?**

**Manual approach (capstone):**
- 1 Secret, 1 ConfigMap, 1 StatefulSet, 1 Deployment, 2 Services (Headless + NodePort), 1 HPA, 1 PVC
- Total: ~8–10 resources

**Helm approach (bitnami/wordpress):**
- Multiple Deployments (WordPress, MariaDB), multiple Services (LoadBalancer, ClusterIP, Headless), multiple Secrets, multiple ConfigMaps, 2+ PVCs (8Gi for MariaDB, 10Gi for WordPress)
- Total: 15–20+ resources

**Control comparison:**
- Manual approach: Maximum control, every component is customized
- Helm approach: Less control but more convenience — pre-configured best practices, easier upgrades, multiple components
- Manual offers: Complete understanding, granular control, no vendor lock-in

**Q2: What are the advantages of using Helm for WordPress deployment?**

- **One-command deployment:** `helm install` deploys the entire stack
- **Parameterization:** Values file for configuration (no manifest editing)
- **Dependency management:** Handles MySQL, Redis, etc.
- **Upgrades/Rollbacks:** Easy version management
- **Reusability:** Same chart for different environments
- **Configuration:** Override any setting via `values.yaml`
- **Maintenance:** Community maintains best practices
- **Multi-component:** Manages related services together

---

## Task 8: Clean Up and Reflect

### Final Verification
```bash
# Take final look
kubectl get all -n capstone
kubectl get pvc -n capstone
kubectl get hpa -n capstone
```

### Delete Namespace
```bash
# Delete the namespace
kubectl delete namespace capstone

# Verify deletion
kubectl get namespaces

# Reset default namespace
kubectl config set-context --current --namespace=default

# Verify everything is removed
kubectl get all -n capstone
```

**Expected Output:**
```
namespace "capstone" deleted
Context "docker-desktop" modified.
No resources found in capstone namespace.
```

**Q1: Did deleting the namespace remove everything? Why?**

Yes, deleting the namespace removes everything because:
- **All resources are namespace-scoped:** All created resources (pods, services, deployments, statefulsets, configmaps, secrets, PVCs, HPA) exist within the `capstone` namespace
- **Cascade deletion:** Namespace deletion triggers garbage collection
- **All resources removed:** Every resource in the namespace is deleted
- **Cluster-scoped exceptions:** ClusterRole, ClusterRoleBinding, StorageClass (if created) persist, but these weren't created in this capstone
- **Automatic cleanup:** Kubernetes controller manager handles cleanup of remaining resources

**Q2: List all 12 Kubernetes concepts used in this capstone.**

1. **Namespace** – Resource isolation (`capstone`)
2. **Secret** – MySQL credentials (`mysql-secret`)
3. **ConfigMap** – WordPress configuration (`wordpress-config`)
4. **PersistentVolumeClaim (PVC)** – MySQL data persistence (from `volumeClaimTemplates`)
5. **StatefulSet** – MySQL management with stable identities
6. **Headless Service** – MySQL service without load balancing
7. **Deployment** – WordPress pod management
8. **NodePort Service** – External WordPress access
9. **Resource Limits** – CPU and memory constraints
10. **Probes** – Liveness and readiness checks
11. **HorizontalPodAutoscaler (HPA)** – Auto-scaling WordPress
12. **Helm** – Bonus comparison with package manager

---

## Complete Working YAML Files

### 1. `mysql-secret.yaml`
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: capstone
type: Opaque
stringData:
  MYSQL_ROOT_PASSWORD: rootpassword123
  MYSQL_DATABASE: wordpress
  MYSQL_USER: wpuser
  MYSQL_PASSWORD: wppassword123
```

### 2. `mysql-service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: capstone
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
```

### 3. `mysql-statefulset.yaml`
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: capstone
spec:
  serviceName: mysql
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        envFrom:
        - secretRef:
            name: mysql-secret
        ports:
        - containerPort: 3306
        resources:
          requests:
            cpu: 250m
            memory: 512Mi
          limits:
            cpu: 500m
            memory: 1Gi
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

### 4. `wordpress-configmap.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: wordpress-config
  namespace: capstone
data:
  WORDPRESS_DB_HOST: mysql-0.mysql.capstone.svc.cluster.local:3306
  WORDPRESS_DB_NAME: wordpress
```

### 5. `wordpress-deployment.yaml` (WORKING VERSION)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  namespace: capstone
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress:6.4.3
        ports:
        - containerPort: 80
        env:
        - name: WORDPRESS_DB_HOST
          value: mysql-0.mysql.capstone.svc.cluster.local:3306
        - name: WORDPRESS_DB_NAME
          value: wordpress
        - name: WORDPRESS_DB_USER
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_USER
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_PASSWORD
        resources:
          requests:
            cpu: 100m
            memory: 256Mi
          limits:
            cpu: 200m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /wp-admin/install.php
            port: 80
          initialDelaySeconds: 120
          periodSeconds: 10
          failureThreshold: 5
        readinessProbe:
          httpGet:
            path: /wp-admin/install.php
            port: 80
          initialDelaySeconds: 90
          periodSeconds: 5
          failureThreshold: 3
```

### 6. `wordpress-service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  namespace: capstone
spec:
  type: NodePort
  selector:
    app: wordpress
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

### 7. `wordpress-hpa.yaml`
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: wordpress-hpa
  namespace: capstone
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: wordpress
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

---

## Screenshots

> **Where to attach:** Put this whole section right here — directly under the "Complete Working YAML Files" block and before you wrap up the document. Save each screenshot into a `screenshots/` folder next to this `.md` file, named exactly as below, and the image links will render automatically. If you'd rather store them elsewhere, just update the relative path in each `![...](path)` line.

### Screenshot 1: WordPress Blog Post
- **File:** `screenshots/screenshot-1-wordpress-blog.png`
- Show the published blog post in browser, URL showing `http://localhost:8080`, title "Kubernetes Capstone Day 60", and the post content.



### Screenshot 2: `kubectl get all -n capstone`

<img width="1808" height="834" alt="image" src="https://github.com/user-attachments/assets/3fff5a26-569f-4e4d-bcdb-c459e0ed365a" />


### Screenshot 3: `kubectl get hpa -n capstone`

<img width="740" height="71" alt="image" src="https://github.com/user-attachments/assets/63c99177-3f43-4b5a-8c17-2b8b0b36dcff" />


### Screenshot 4: `kubectl get pvc -n capstone`

<img width="923" height="105" alt="image" src="https://github.com/user-attachments/assets/4f0a9abf-3973-44d0-a505-a4ae128ba286" />


### Screenshot 5: WordPress Admin Dashboard

<img width="3324" height="1730" alt="image" src="https://github.com/user-attachments/assets/7d4a20da-e8c9-4057-bcea-d90cc9f3d9a9" />



### Screenshot 6: Self-Healing Verification

<img width="1726" height="562" alt="image" src="https://github.com/user-attachments/assets/5ebf283d-8116-4792-9808-0d28a2cab5f2" />


### Screenshot 7: Helm Comparison

<img width="840" height="308" alt="image" src="https://github.com/user-attachments/assets/bfa86f71-2731-47d7-95df-8dec0a686515" />


<img width="1808" height="834" alt="image" src="https://github.com/user-attachments/assets/bae903d0-675c-4897-9441-d0753f0c7a8a" />


