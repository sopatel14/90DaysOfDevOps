# Day 51 – Kubernetes Manifests and Your First Pods

> Part of my #90DaysOfDevOps journey

---

# Objective

Today I learned how Kubernetes resources are defined using YAML manifests and deployed my first Pods. I created Pod manifests from scratch, explored Pods using `kubectl`, practiced both declarative and imperative approaches, worked with labels, and understood why standalone Pods are rarely used in production.

---

# What is a Kubernetes Manifest?

A Kubernetes manifest is a YAML file that describes the desired state of a Kubernetes resource. When applied using `kubectl`, Kubernetes compares the desired state with the current state and makes the necessary changes.

Every Kubernetes resource begins with four important top-level fields.

---

# The Four Required Fields

## 1. apiVersion

Specifies which Kubernetes API version should be used for the resource.

Example:

```yaml
apiVersion: v1
```

For Pods, the API version is `v1`.

---

## 2. kind

Defines the type of Kubernetes resource.

Examples include:

- Pod
- Deployment
- Service
- ConfigMap
- Secret

Example:

```yaml
kind: Pod
```

---

## 3. metadata

Provides identifying information about the resource.

Common fields include:

- name
- labels
- namespace
- annotations

Example:

```yaml
metadata:
  name: nginx-pod
  labels:
    app: nginx
```

---

## 4. spec

Describes the desired state of the resource.

For a Pod, this includes:

- Containers
- Images
- Ports
- Commands
- Volumes
- Environment variables

Example:

```yaml
spec:
  containers:
  - name: nginx
    image: nginx:latest
```

---

# Task 1 – Nginx Pod

## nginx-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

## Commands Used

```bash
kubectl apply -f nginx-pod.yaml

kubectl get pods

kubectl get pods -o wide

kubectl describe pod nginx-pod

kubectl logs nginx-pod

kubectl exec -it nginx-pod -- /bin/bash

curl localhost:80
```

### Result

The Pod successfully entered the **Running** state, and accessing `localhost:80` from inside the container displayed the default Nginx welcome page.

---

# Task 2 – BusyBox Pod

## busybox-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
  labels:
    app: busybox
    environment: dev
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command:
      - sh
      - -c
      - echo Hello from BusyBox && sleep 3600
```

## Commands Used

```bash
kubectl apply -f busybox-pod.yaml

kubectl get pods

kubectl logs busybox-pod
```

### Result

The custom command printed:

```
Hello from BusyBox
```

The container remained running because of the `sleep 3600` command.

---

# Task 3 – Third Pod with Labels

## third-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: alpine-pod
  labels:
    app: alpine
    environment: test
    team: devops
spec:
  containers:
  - name: alpine
    image: alpine:latest
    command:
      - sh
      - -c
      - sleep 3600
```

---

# Labels Used

| Label | Value |
|--------|-------|
| app | alpine |
| environment | test |
| team | devops |

---

# Filtering Pods Using Labels

```bash
kubectl get pods --show-labels

kubectl get pods -l app=nginx

kubectl get pods -l environment=dev

kubectl get pods -l team=devops
```

Labels make it easy to organize and select Kubernetes resources.

---

# Imperative vs Declarative

## Imperative Approach

Commands directly create resources.

Example:

```bash
kubectl run redis-pod --image=redis:latest
```

### Advantages

- Quick
- Useful for testing
- No YAML required

### Disadvantages

- Hard to reproduce
- Not suitable for Infrastructure as Code

---

## Declarative Approach

Resources are defined in YAML files.

Example:

```bash
kubectl apply -f nginx-pod.yaml
```

### Advantages

- Version controlled
- Easy to review
- Repeatable
- Preferred in production

---

# Dry Run Validation

## Client Side

```bash
kubectl apply -f nginx-pod.yaml --dry-run=client
```

## Server Side

```bash
kubectl apply -f nginx-pod.yaml --dry-run=server
```

Dry-run validates manifests before creating resources.

---

# Generated Manifest

Using:

```bash
kubectl run test-pod \
--image=nginx \
--dry-run=client \
-o yaml
```

Kubernetes automatically generated a Pod manifest which can be customized before deployment.

---

# Pod Exploration Commands

```bash
kubectl get pods

kubectl get pods -o wide

kubectl describe pod nginx-pod

kubectl logs nginx-pod

kubectl exec -it nginx-pod -- /bin/bash

kubectl get pods --show-labels
```

---

# What Happens When a Standalone Pod is Deleted?

Deleting a standalone Pod permanently removes it from the cluster.

Unlike Deployments or ReplicaSets, a standalone Pod has no controller monitoring it, so Kubernetes does **not** recreate it automatically.

This is why production workloads are typically managed using Deployments instead of individual Pods.

---

# Key Learnings

- Learned the anatomy of Kubernetes manifests.
- Wrote three Pod manifests manually.
- Understood the purpose of apiVersion, kind, metadata, and spec.
- Created Pods using declarative YAML files.
- Compared imperative and declarative resource creation.
- Explored Pods using `kubectl describe`, `logs`, and `exec`.
- Learned how labels organize Kubernetes resources.
- Practiced label filtering and modification.
- Understood why standalone Pods are not used in production.

---

# Screenshots

## Screenshot 1

`kubectl get pods`

<img width="743" height="111" alt="image" src="https://github.com/user-attachments/assets/74ef9493-ffbc-4016-acc7-c94ad39cdad2" />


---

## Screenshot 2

`kubectl get pods --show-labels`

<img width="739" height="103" alt="image" src="https://github.com/user-attachments/assets/b9bbc04c-8adf-48c5-9eb1-1decad5bbb1a" />

---

## Screenshot 3

`kubectl logs busybox-pod`

<img width="746" height="73" alt="image" src="https://github.com/user-attachments/assets/dc393b27-b039-4f55-a7d0-a3222fae988f" />

---

## Screenshot 4

`kubectl exec -it nginx-pod -- /bin/bash`

followed by

```bash
curl localhost:80
```

<img width="1902" height="1020" alt="image" src="https://github.com/user-attachments/assets/2348855f-72b3-4852-90fe-f30f29a16637" />


---

## Screenshot 5

`kubectl get pods -o wide`

<img width="830" height="113" alt="image" src="https://github.com/user-attachments/assets/46c51d5c-01b0-4229-831a-a5d789c1ab24" />



---
