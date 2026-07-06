# Day 52 – Kubernetes Namespaces and Deployments

> Part of my #90DaysOfDevOps journey

---

# Objective

Today I learned how Kubernetes organizes resources using **Namespaces** and how **Deployments** provide self-healing, scaling, and rolling updates for applications. Unlike standalone Pods, Deployments ensure the desired number of replicas are always running, making them the preferred way to manage workloads in production.

---

# What are Namespaces?

Namespaces are virtual clusters within a Kubernetes cluster that help organize and isolate resources.

They are commonly used to:

- Separate development, staging, and production environments
- Isolate teams and applications
- Avoid naming conflicts
- Apply resource quotas and access controls

---

# Default Kubernetes Namespaces

| Namespace | Purpose |
|------------|---------|
| default | Default namespace for user-created resources |
| kube-system | Kubernetes control plane and system components |
| kube-public | Publicly readable cluster information |
| kube-node-lease | Stores node heartbeat information |

---

# Custom Namespaces Created

## Development

```bash
kubectl create namespace dev
```

## Staging

```bash
kubectl create namespace staging
```

## Production (Manifest)

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

Apply:

```bash
kubectl apply -f namespace.yaml
```

---

# Running Pods in Different Namespaces

Development:

```bash
kubectl run nginx-dev --image=nginx:latest -n dev
```

Staging:

```bash
kubectl run nginx-staging --image=nginx:latest -n staging
```

View all Pods:

```bash
kubectl get pods -A
```

---

# Deployment Manifest

## nginx-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
  labels:
    app: nginx

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx
        image: nginx:1.24
        ports:
        - containerPort: 80
```

---

# Deployment Manifest Breakdown

## apiVersion

```yaml
apps/v1
```

API version used for Deployments.

---

## kind

```yaml
Deployment
```

Defines the Kubernetes resource type.

---

## metadata

Contains:

- Deployment name
- Namespace
- Labels

---

## replicas

```yaml
replicas: 3
```

Tells Kubernetes to maintain three identical Pods.

---

## selector

```yaml
selector:
  matchLabels:
    app: nginx
```

Links the Deployment to the Pods it manages.

---

## template

Defines the Pod blueprint used whenever Kubernetes creates a new Pod.

---

# Deployment Verification

Commands used:

```bash
kubectl get deployments -n dev

kubectl get pods -n dev

kubectl describe deployment nginx-deployment -n dev
```

---

# Deployment Status Columns

| Column | Meaning |
|----------|---------|
| READY | Running replicas / Desired replicas |
| UP-TO-DATE | Pods using the latest Deployment configuration |
| AVAILABLE | Pods currently available to serve traffic |

---

# Self-Healing Demonstration

Deleted one Pod:

```bash
kubectl delete pod <pod-name> -n dev
```

Immediately checked:

```bash
kubectl get pods -n dev
```

## Observation

The deleted Pod was automatically replaced with a **new Pod having a different name**.

This demonstrates Kubernetes' self-healing capability through the Deployment controller.

---

# Scaling the Deployment

## Scale Up

```bash
kubectl scale deployment nginx-deployment --replicas=5 -n dev
```

Result:

- Kubernetes created two additional Pods.

---

## Scale Down

```bash
kubectl scale deployment nginx-deployment --replicas=2 -n dev
```

Result:

- Kubernetes terminated the extra Pods while keeping the desired number running.

---

# Declarative Scaling

Modified:

```yaml
replicas: 4
```

Then applied:

```bash
kubectl apply -f nginx-deployment.yaml
```

Kubernetes reconciled the current state with the desired state automatically.

---

# Imperative vs Declarative Scaling

## Imperative

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

Fast and useful for testing.

---

## Declarative

Edit YAML:

```yaml
replicas: 5
```

Then:

```bash
kubectl apply -f nginx-deployment.yaml
```

Preferred in production because it is version-controlled and repeatable.

---

# Rolling Update

Updated the container image:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.25 -n dev
```

Checked rollout status:

```bash
kubectl rollout status deployment/nginx-deployment -n dev
```

---

## What Happened?

Kubernetes replaced Pods one at a time.

Old Pods were terminated only after new Pods became healthy, ensuring zero downtime.

---

# Rollout History

```bash
kubectl rollout history deployment/nginx-deployment -n dev
```

Shows previous Deployment revisions.

---

# Rollback

```bash
kubectl rollout undo deployment/nginx-deployment -n dev
```

Verified:

```bash
kubectl describe deployment nginx-deployment -n dev
```

The Deployment successfully reverted to the previous Nginx image version.

---

# Deployment vs Standalone Pod

| Standalone Pod | Deployment |
|----------------|------------|
| Runs independently | Managed by Deployment controller |
| Deleted permanently | Automatically recreated |
| No scaling | Supports scaling |
| No rolling updates | Supports rolling updates |
| Not recommended for production | Recommended for production |

---

# Commands Practiced

```bash
kubectl get namespaces

kubectl get pods -A

kubectl get deployments

kubectl scale deployment

kubectl rollout status

kubectl rollout history

kubectl rollout undo

kubectl delete deployment

kubectl delete namespace
```

---

# Key Learnings

- Learned why Namespaces are important.
- Created custom namespaces.
- Deployed applications inside specific namespaces.
- Created my first Kubernetes Deployment.
- Learned how Deployments maintain desired state.
- Demonstrated Kubernetes self-healing.
- Performed imperative and declarative scaling.
- Executed rolling updates with zero downtime.
- Rolled back to a previous application version.
- Understood why Deployments are used instead of standalone Pods in production.

---

# Screenshots

## Screenshot 1

`kubectl get namespaces`

<img width="965" height="189" alt="image" src="https://github.com/user-attachments/assets/c003aa2f-4aba-4f9a-97a5-64cf31d3ed0d" />

---

## Screenshot 2

`kubectl get deployments -n dev`

<img width="964" height="79" alt="image" src="https://github.com/user-attachments/assets/ba4fe453-c583-470c-934b-e03cffab84b1" />

---

## Screenshot 3

`kubectl get pods -n dev`

<img width="966" height="115" alt="image" src="https://github.com/user-attachments/assets/778ae1c5-be3a-4b53-ad2f-c57c1fe93b48" />

---

## Screenshot 4

`kubectl get pods -A`

<img width="972" height="343" alt="image" src="https://github.com/user-attachments/assets/cb922897-e9bc-4573-827c-a2d4599d9eb7" />

---

## Screenshot 5

Scaling Deployment

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

<img width="1916" height="410" alt="image" src="https://github.com/user-attachments/assets/d4794a39-c359-4465-a7ae-6b999c9b112b" />

---

## Screenshot 6

Rolling Update

```bash
kubectl rollout status deployment/nginx-deployment
```

<img width="1934" height="430" alt="image" src="https://github.com/user-attachments/assets/87d38e6e-1de3-402d-b011-143db5daca37" />

---

# Conclusion

Today's session highlighted why Deployments are the standard way to run applications in Kubernetes. They provide automatic recovery, scaling, rolling updates, and rollbacks, making application management more reliable and resilient compared to standalone Pods.
