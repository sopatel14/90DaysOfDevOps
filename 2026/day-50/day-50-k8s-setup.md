# Day 50 – Kubernetes Architecture and Cluster Setup

---

# Objective

Today marks the beginning of my Kubernetes journey. After learning how to build and run containers using Docker, I explored why Kubernetes exists, understood its architecture, created my first local Kubernetes cluster, and practiced basic `kubectl` commands.

---

# Task 1 – Kubernetes Story

## Why was Kubernetes created?

Docker makes it easy to package and run applications inside containers, but managing hundreds or thousands of containers across multiple servers becomes difficult.

Kubernetes solves this problem by automating:

- Container deployment
- Scaling applications
- Self-healing failed containers
- Load balancing
- Service discovery
- Rolling updates and rollbacks
- Resource management

Instead of manually managing containers on different machines, Kubernetes manages the entire cluster automatically.

---

## Who created Kubernetes?

Kubernetes was originally created by **Google**.

It was inspired by Google's internal container orchestration platform called **Borg**, which Google had been using for many years to manage large-scale workloads.

Later, Kubernetes was donated to the **Cloud Native Computing Foundation (CNCF)** where it is now maintained as an open-source project.

---

## What does Kubernetes mean?

The word **Kubernetes** comes from Greek and means:

> **"Helmsman"** or **"Ship Pilot"**

It represents steering and managing containerized applications.

Its abbreviation **K8s** replaces the eight letters between **K** and **s**.

---

# Verification

After reviewing the official Kubernetes documentation, my understanding matched the following:

- Kubernetes automates container orchestration.
- It originated from Google's Borg project.
- It is maintained by CNCF.
- Kubernetes means "Helmsman."

---

# Task 2 – Kubernetes Architecture

## Kubernetes Architecture Diagram

```text
                    kubectl
                       │
                       ▼
               +----------------+
               |   API Server   |
               +----------------+
                       │
      ┌────────────────┼────────────────┐
      ▼                ▼                ▼
+------------+   +------------+   +----------------+
| Scheduler  |   | Controller |   |     etcd       |
|            |   |  Manager   |   | Cluster State  |
+------------+   +------------+   +----------------+
                       │
                       ▼
               Worker Node(s)
+------------------------------------------------------+
| kubelet                                              |
|                                                      |
| kube-proxy                                           |
|                                                      |
| Container Runtime (containerd / CRI-O)               |
|                                                      |
| Pods                                                 |
+------------------------------------------------------+
```

---

## Control Plane Components

### API Server

- Entry point for every Kubernetes request.
- All `kubectl` commands communicate with the API Server.

---

### etcd

- Distributed key-value database.
- Stores the entire cluster state.
- Holds information about nodes, pods, deployments, secrets, and configurations.

---

### Scheduler

- Watches for newly created pods.
- Chooses the most suitable worker node based on available resources and scheduling policies.

---

### Controller Manager

Runs multiple controllers that continuously monitor the cluster and ensure the actual state matches the desired state.

Examples include:

- Node Controller
- Deployment Controller
- ReplicaSet Controller
- Job Controller

---

## Worker Node Components

### kubelet

- Agent running on every worker node.
- Receives instructions from the API Server.
- Creates and manages Pods.

---

### kube-proxy

- Handles networking.
- Maintains networking rules.
- Enables communication between Pods and Services.

---

### Container Runtime

Actually runs the containers.

Examples:

- containerd
- CRI-O

---

# What happens when I run?

```bash
kubectl apply -f pod.yaml
```

Flow:

1. kubectl sends the request to the API Server.
2. API Server validates the request.
3. Desired state is stored inside etcd.
4. Scheduler selects the best worker node.
5. kubelet on that node receives the instruction.
6. kubelet asks the container runtime to create the Pod.
7. kube-proxy updates networking if required.
8. Pod starts running.
9. Cluster state is updated.

---

## What happens if the API Server goes down?

- No new changes can be made.
- kubectl commands fail.
- Existing workloads continue running.
- Controllers cannot reconcile changes until the API Server is available again.

---

## What happens if a Worker Node goes down?

- Controller Manager detects the node failure.
- Pods on that node become unavailable.
- Scheduler places replacement Pods on healthy worker nodes.
- Kubernetes automatically attempts to restore the desired state.

---

# Task 3 – Install kubectl

## Installation

Since I am using macOS:

```bash
brew install kubectl
```

Verify:

```bash
kubectl version --client
```


---

## Screenshot

<img width="565" height="132" alt="image" src="https://github.com/user-attachments/assets/5d358baf-9a9c-4a0a-8103-dfd2514f0aff" />


---

# Task 4 – Local Kubernetes Cluster

## Tool Chosen

✅ **kind (Kubernetes IN Docker)**

### Why I chose kind

- Lightweight
- Fast startup
- Uses Docker containers instead of virtual machines
- Perfect for local development
- Easy to recreate clusters

---

## Create Cluster

```bash
kind create cluster --name devops-cluster
```

---

## Verify

```bash
kubectl cluster-info

kubectl get nodes
```

---

## Screenshot

<img width="565" height="84" alt="image" src="https://github.com/user-attachments/assets/fe94c3f7-8cf7-41d5-a47a-49d57f7c0ad7" />


---

# Task 5 – Explore the Cluster

## Cluster Information

```bash
kubectl cluster-info
```

---

## List Nodes

```bash
kubectl get nodes
```

---

## Node Details

```bash
kubectl describe node <node-name>
```

---

## Namespaces

```bash
kubectl get namespaces
```

---

## View All Pods

```bash
kubectl get pods -A
```

---

## kube-system Pods

```bash
kubectl get pods -n kube-system
```

---

## Screenshot

<img width="744" height="211" alt="image" src="https://github.com/user-attachments/assets/d7fb754f-f3c2-4a16-9af9-3369c30c4e04" />


---

# kube-system Components

| Pod | Purpose |
|------|----------|
| kube-apiserver | Entry point for all cluster communication |
| etcd | Stores cluster state and configuration |
| kube-scheduler | Assigns Pods to worker nodes |
| kube-controller-manager | Ensures desired state matches actual state |
| kube-proxy | Handles cluster networking |
| CoreDNS | Provides DNS resolution inside the cluster |

---

# Matching Architecture with Running Pods

| Architecture Component | Running Pod |
|------------------------|-------------|
| API Server | kube-apiserver |
| Scheduler | kube-scheduler |
| Controller Manager | kube-controller-manager |
| etcd | etcd |
| kube-proxy | kube-proxy |
| DNS | CoreDNS |

---

# Task 6 – Cluster Lifecycle

## Delete Cluster

```bash
kind delete cluster --name devops-cluster
```

---

## Recreate Cluster

```bash
kind create cluster --name devops-cluster
```

---

## Verify

```bash
kubectl get nodes
```
<img width="1488" height="268" alt="image" src="https://github.com/user-attachments/assets/5852b5d4-c56d-4dba-b621-55bcb12af745" />


---

## Current Context

```bash
kubectl config current-context
```

---

## List Contexts

```bash
kubectl config get-contexts
```

---

## View kubeconfig

```bash
kubectl config view
```

---

# What is kubeconfig?

The **kubeconfig** file stores information about Kubernetes clusters, users, authentication credentials, and contexts.

It tells `kubectl` which cluster to communicate with and how to authenticate.

### Default Location

```text
~/.kube/config
```

---

# Commands Practiced

```bash
kubectl version --client

kubectl cluster-info

kubectl get nodes

kubectl describe node

kubectl get namespaces

kubectl get pods -A

kubectl get pods -n kube-system

kubectl config current-context

kubectl config get-contexts

kubectl config view

kind create cluster --name devops-cluster

kind delete cluster --name devops-cluster
```

---

# Key Learnings

- Learned why Kubernetes was created.
- Understood the Control Plane and Worker Node architecture.
- Learned how requests flow through Kubernetes.
- Installed `kubectl`.
- Created my first Kubernetes cluster using **kind**.
- Explored namespaces, nodes, and system Pods.
- Practiced deleting and recreating a Kubernetes cluster.
- Learned how `kubeconfig` stores cluster connection details.

---
