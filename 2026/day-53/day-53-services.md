# Day 53 – Kubernetes Services

Today I learned how Kubernetes **Services** provide a stable network endpoint for Pods.

Pods are ephemeral and receive new IP addresses whenever they are recreated. Instead of communicating directly with Pod IPs, Kubernetes Services provide a stable IP address and DNS name while automatically load-balancing traffic across all healthy Pods.

---

# Why Kubernetes Services?

Every Pod receives its own IP address.

However, there are two major problems:

- Pod IPs change whenever Pods restart.
- Deployments create multiple Pods, making it difficult to know which Pod to connect to.

A **Service** solves these problems by providing:

- A stable virtual IP (ClusterIP)
- Automatic DNS name
- Built-in load balancing across matching Pods

```
Client
   │
   ▼
Kubernetes Service
   │
 ┌─┴───────────────┐
 ▼                 ▼
Pod 1           Pod 2
                    ▼
                 Pod 3
```

---

# Lab Environment

- Kubernetes Cluster
- kubectl
- Nginx Deployment
- BusyBox Test Pod

---

# Task 1 – Deploy the Application

Created an Nginx Deployment with 3 replicas.

### Deployment Manifest - app-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80


```

### Commands Used

```bash
kubectl apply -f app-deployment.yaml

kubectl get pods -o wide
```

### Verification

✔ Deployment created successfully

✔ Three Pods running

✔ Each Pod received its own IP address

---

## 📸 Screenshot

<img width="1772" height="290" alt="image" src="https://github.com/user-attachments/assets/d71d8c32-0569-48b2-a564-26970cf7804b" />


---

# Task 2 – ClusterIP Service

Created a ClusterIP Service to expose the Deployment internally.

### Service Manifest - clusterip-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-clusterip
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
```

### Apply Service

```bash
kubectl apply -f clusterip-service.yaml

kubectl get services
```

ClusterIP is the default Service type.

It can only be accessed from inside the cluster.

---

## Testing from Inside the Cluster

```bash
kubectl run test-client \
--image=busybox:latest \
--rm -it \
--restart=Never -- sh
```

Inside BusyBox:

```bash
wget -qO- http://web-app-clusterip
```

The Nginx welcome page was successfully returned.

---

## 📸 Screenshot

<img width="1764" height="844" alt="image" src="https://github.com/user-attachments/assets/6afc836f-ede1-4ac4-a93d-a8b20cba19f6" />


---

# Task 3 – Kubernetes DNS

Every Service automatically receives a DNS name.

Short name

```
web-app-clusterip
```

Full DNS

```
web-app-clusterip.default.svc.cluster.local
```

### DNS Test

```bash
nslookup web-app-clusterip
```

The returned IP matched the ClusterIP assigned to the Service.

---

## 📸 Screenshot

<img width="1766" height="238" alt="image" src="https://github.com/user-attachments/assets/2191ffb2-8a43-431b-a7b2-64143bfb83ab" />


<img width="2052" height="572" alt="image" src="https://github.com/user-attachments/assets/31019178-1be8-40d6-adc3-baecc58950d2" />


---

# Task 4 – NodePort Service

Created a NodePort Service to expose the application outside the cluster.

### Service Manifest

```yaml
# nodeport-service.yaml
```

### Apply

```bash
kubectl apply -f nodeport-service.yaml
```

Verify

```bash
kubectl get services
```

Access

```
http://localhost:30080
```

(or NodeIP:30080 depending on the environment)

Successfully accessed the Nginx welcome page.

---

## 📸 Screenshot

<img width="2046" height="1074" alt="image" src="https://github.com/user-attachments/assets/1dc6fe5e-e7ec-48f4-a327-c60680812a61" />


---

# Task 5 – LoadBalancer Service

Created a LoadBalancer Service.

### Service Manifest

```yaml
# loadbalancer-service.yaml
```

Apply

```bash
kubectl apply -f loadbalancer-service.yaml
```

Check

```bash
kubectl get services
```

Since this lab was performed on a local Kubernetes cluster, the External IP remained:

```
<pending>
```

This is expected because local clusters do not provision cloud load balancers.

---

## 📸 Screenshot


<img width="1025" height="121" alt="image" src="https://github.com/user-attachments/assets/0aa1d2f7-8dd6-4c02-82a8-017e1c0e2594" />


---

# Task 6 – Compare Service Types

```bash
kubectl get services -o wide
```

| Service Type | Accessible From | Use Case |
|--------------|----------------|----------|
| ClusterIP | Inside Cluster | Internal communication |
| NodePort | NodeIP:Port | Development & Testing |
| LoadBalancer | Public IP | Production Cloud Access |

---

## Service Hierarchy

```
LoadBalancer
      │
      ▼
 NodePort
      │
      ▼
 ClusterIP
      │
      ▼
 Pods
```

A LoadBalancer Service automatically creates:

- ClusterIP
- NodePort
- External Load Balancer (Cloud Only)

---

### Verify

```bash
kubectl describe service web-app-loadbalancer
```

Observed:

- ClusterIP ✔
- NodePort ✔
- LoadBalancer configuration ✔

---

## 📸 Screenshot


<img width="1664" height="852" alt="image" src="https://github.com/user-attachments/assets/08f8c4c4-d574-44ef-bb76-f821cf1cd798" />


---

# Understanding Kubernetes DNS

Every Service gets an automatically generated DNS entry.

Format

```
<ServiceName>.<Namespace>.svc.cluster.local
```

Example

```
web-app-clusterip.default.svc.cluster.local
```

Benefits

- No need to remember Pod IPs
- Services continue working after Pod replacement
- Easier communication between applications

---

# Understanding Endpoints

Services do not directly connect to Pods.

Instead, Kubernetes creates an Endpoint object that stores the IP addresses of all matching Pods.

View Endpoints

```bash
kubectl get endpoints
```

Detailed View

```bash
kubectl describe endpoints web-app-clusterip
```

Endpoints update automatically whenever Pods are created or deleted.

---

# Useful Commands

```bash
kubectl get services

kubectl get services -o wide

kubectl describe service web-app-clusterip

kubectl get endpoints

kubectl describe endpoints web-app-clusterip

kubectl get pods -o wide
```

---

# Cleanup

```bash
kubectl delete -f app-deployment.yaml

kubectl delete -f clusterip-service.yaml

kubectl delete -f nodeport-service.yaml

kubectl delete -f loadbalancer-service.yaml
```

Verify

```bash
kubectl get pods

kubectl get services
```

Only the default Kubernetes Service remained.

---

# Key Learnings

- Pods have temporary IP addresses.
- Services provide a stable network endpoint.
- ClusterIP is used for internal communication.
- NodePort exposes applications using a node port.
- LoadBalancer is used in cloud environments.
- Kubernetes automatically provides DNS for every Service.
- Services route traffic using Pod labels.
- Endpoints maintain the list of healthy Pods behind a Service.

---

# Files Created

```
app-deployment.yaml

clusterip-service.yaml

nodeport-service.yaml

loadbalancer-service.yaml

day-53-services.md
```

---
