# Day 58 – Metrics Server and Horizontal Pod Autoscaler (HPA)

## Overview

On Day 58, I learned how Kubernetes automatically scales applications based on resource utilization using the **Metrics Server** and **Horizontal Pod Autoscaler (HPA)**.

The Metrics Server collects CPU and memory usage from every node and pod, while HPA uses those metrics to automatically increase or decrease the number of application replicas based on demand.

---

# Objectives

- Install and verify Metrics Server
- Explore real-time resource usage using `kubectl top`
- Deploy an application with CPU requests
- Create an HPA using the imperative command
- Generate load and observe automatic scaling
- Create an HPA using the declarative YAML approach
- Understand HPA scaling behavior
- Clean up resources

---

# What is Metrics Server?

Metrics Server is a lightweight cluster-wide component that collects CPU and memory usage from kubelets running on every node.

Unlike Prometheus, it is **not meant for long-term monitoring**.

Its primary purpose is to provide resource metrics to Kubernetes components such as:

- Horizontal Pod Autoscaler (HPA)
- Vertical Pod Autoscaler (VPA)
- `kubectl top`

Without Metrics Server:

- `kubectl top` does not work
- HPA cannot determine CPU utilization
- Autoscaling is impossible

---

# Why HPA Needs Metrics Server

The Horizontal Pod Autoscaler continuously checks CPU (or memory) utilization.

It gets those metrics from the Metrics Server every **15 seconds**.

Based on those values, Kubernetes decides whether to:

- Increase replicas
- Keep replicas unchanged
- Reduce replicas

Without Metrics Server, HPA shows:

```
TARGETS: <unknown>
```

because it has no metrics to calculate utilization.

---

# How HPA Calculates Desired Replicas

HPA uses the following formula:

```
desiredReplicas =
ceil(
currentReplicas ×
(currentUsage / targetUsage)
)
```

Example:

Current replicas = 2

Current CPU utilization = 100%

Target CPU utilization = 50%

```
Desired Replicas

= ceil(2 × 100/50)

= ceil(4)

= 4
```

Kubernetes automatically scales the Deployment to **4 replicas**.

---

# autoscaling/v1 vs autoscaling/v2

| Feature | autoscaling/v1 | autoscaling/v2 |
|----------|----------------|----------------|
| CPU metrics | ✅ | ✅ |
| Memory metrics | ❌ | ✅ |
| Custom metrics | ❌ | ✅ |
| External metrics | ❌ | ✅ |
| Scaling behavior | ❌ | ✅ |
| Multiple metrics | ❌ | ✅ |

The `autoscaling/v2` API provides much finer control over scaling policies and behavior.

---

# Task 1 – Install Metrics Server

## Verify Metrics Server

```bash
kubectl get pods -n kube-system -l k8s-app=metrics-server
```

Output

```text
metrics-server-55bf4495db-c9wk7   1/1   Running
```

## Verify Resource Metrics

```bash
kubectl top nodes
```

Output

```text
NAME                 CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)
kind-control-plane   217m         1%       1093Mi          13%
kind-worker          42m          0%       273Mi           3%
kind-worker2         48m          0%       307Mi           3%
```

### Verification Answer

**Q. What is the current CPU and memory usage of your node?**

**Answer:**

| Node | CPU | Memory |
|------|------|---------|
| kind-control-plane | **217m (1%)** | **1093Mi (13%)** |
| kind-worker | **42m (0%)** | **273Mi (3%)** |
| kind-worker2 | **48m (0%)** | **307Mi (3%)** |

---

📸 **Screenshot Placeholder**

<img width="662" height="273" alt="image" src="https://github.com/user-attachments/assets/f5893ec0-63af-4d3b-ad6c-ef39c4e2770a" />

<img width="1138" height="410" alt="image" src="https://github.com/user-attachments/assets/0bc2ab90-0f46-4963-8372-2a96c275a65b" />


---

# Task 2 – Explore kubectl top

Commands

```bash
kubectl top nodes
kubectl top pods -A
kubectl top pods -A --sort-by=cpu
```

Top CPU Consumers

```text
kube-apiserver-kind-control-plane    59m
etcd-kind-control-plane              32m
kube-controller-manager              31m
```

### Verification Answer

**Q. Which pod is using the most CPU right now?**

**Answer:**

The **kube-apiserver-kind-control-plane** pod is using the highest CPU at approximately **59m**.

### Key Learning

`kubectl top` displays **actual live CPU and memory usage**, whereas resource requests and limits are only configuration values.

---

📸 **Screenshot Placeholder**

<img width="671" height="310" alt="image" src="https://github.com/user-attachments/assets/1ddcc631-8505-44d2-9e98-05b1bc078f3b" />

---

# Task 3 – Create Deployment with CPU Requests

Deployment image

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: php-apache
  template:
    metadata:
      labels:
        app: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 200m
          limits:
            cpu: 500m
```

CPU Request

```
200m
```

Expose Deployment

```bash
kubectl expose deployment php-apache --port=80
```

Check Usage

```bash
kubectl top pods
```

Output

```text
php-apache-7d4bd5f475-j44mv   1m   8Mi
```

### Verification Answer

**Q. What is the current CPU usage of the Pod?**

**Answer:**

The pod is currently using approximately **1m CPU** and **8Mi memory**.

---

📸 **Screenshot Placeholder**

<img width="1140" height="194" alt="image" src="https://github.com/user-attachments/assets/cd90383a-28e0-452c-b95c-a8a14b089cc3" />


---

# Task 4 – Create HPA (Imperative)

Command

```bash
kubectl autoscale deployment php-apache \
--cpu-percent=50 \
--min=1 \
--max=10
```

Verify

```bash
kubectl get hpa

kubectl describe hpa php-apache
```

Output

```text
TARGETS

cpu: 0% / 50%
```

### Verification Answer

**Q. What does the TARGETS column show?**

**Answer:**

The TARGETS column shows the **current CPU utilization compared to the target utilization**.

Initially it displayed:

```
0% / 50%
```

meaning the application was using **0% CPU** while the target threshold was **50%**.

---

📸 **Screenshot Placeholder**


<img width="1642" height="926" alt="image" src="https://github.com/user-attachments/assets/178939a3-484e-4dc0-a0b9-5bc7df94f685" />


---

# Task 5 – Generate Load and Watch Autoscaling

Load Generator

```bash
kubectl run load-generator \
--image=busybox:1.36 \
--restart=Never \
-- /bin/sh -c \
"while true; do wget -q -O- http://php-apache; done"
```

Watch Scaling

```bash
kubectl get hpa php-apache --watch
```

Observed Scaling

```text
CPU 86% -> 1 Replica

CPU 250% -> 2 Replicas

CPU 232% -> 4 Replicas

CPU 114% -> 5 Replicas

CPU 54% -> 7 Replicas
```

### Verification Answer

**Q. How many replicas did HPA scale to under load?**

**Answer:**

The Horizontal Pod Autoscaler automatically scaled the Deployment from **1 replica to 7 replicas** as CPU utilization exceeded the configured 50% target.

### Observation

As traffic increased:

- CPU utilization increased rapidly.
- HPA continuously recalculated the desired replica count.
- Additional pods were created until CPU utilization began stabilizing.

---

📸 **Screenshot Placeholder**

<img width="1648" height="540" alt="image" src="https://github.com/user-attachments/assets/9571c6c3-4ea0-4466-ac1d-f01c57ae845b" />


---

# Task 6 – Create HPA using YAML (Declarative)

Applied

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      selectPolicy: Max
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      selectPolicy: Max
```


```bash
kubectl apply -f hpa-v2.yaml
```

Verify

```bash
kubectl describe hpa php-apache
```

Behavior

```text
Scale Up

Stabilization Window:
0 seconds

Scale Down

Stabilization Window:
300 seconds
```

### Verification Answer

**Q. What does the behavior section control?**

**Answer:**

The **behavior** section controls how aggressively the HPA scales applications.

It defines:

- Scale-up speed
- Scale-down speed
- Stabilization windows
- Scaling policies
- Percentage or pod-based scaling limits

In this lab:

- Scale Up occurred immediately with **0-second stabilization**.
- Scale Down waited **300 seconds (5 minutes)** before removing replicas to prevent rapid fluctuations.

---

📸 **Screenshot Placeholder**

<img width="1318" height="880" alt="image" src="https://github.com/user-attachments/assets/ca75d87d-3b5a-41b5-ad88-ec27237832cd" />


---

# Task 7 – Clean Up

Commands

```bash
kubectl delete hpa php-apache

kubectl delete service php-apache

kubectl delete deployment php-apache

kubectl delete pod load-generator
```

The Metrics Server was intentionally left installed for future Kubernetes labs.

---

# Key Takeaways

- Metrics Server provides CPU and memory metrics to Kubernetes.
- `kubectl top` displays real-time resource usage.
- HPA requires CPU requests to calculate utilization.
- HPA scales pods automatically based on CPU usage.
- autoscaling/v2 supports advanced scaling behavior and multiple metric types.
- Scale-up is quick, while scale-down uses a stabilization window to avoid frequent scaling events.

---
