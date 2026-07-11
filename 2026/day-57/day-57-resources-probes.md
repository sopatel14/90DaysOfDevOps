# Day 57 – Resource Requests, Limits, and Probes

## Overview

Today I learned how Kubernetes manages application resources and monitors container health using **Resource Requests**, **Resource Limits**, and **Health Probes**.

Without resource configuration, Kubernetes cannot efficiently schedule workloads, and unhealthy containers may continue running unnoticed. By defining requests, limits, and probes, Kubernetes can make smarter scheduling decisions and automatically recover from failures.

---

# Objectives

- Understand Resource Requests and Resource Limits
- Learn Kubernetes Quality of Service (QoS) Classes
- Observe an OOMKilled container
- Understand Pending Pods caused by insufficient resources
- Configure Liveness Probe
- Configure Readiness Probe
- Configure Startup Probe

---

# Kubernetes Resource Management

## What are Resource Requests?

Resource Requests define the **minimum amount of CPU and Memory** a container needs.

The Kubernetes Scheduler uses requests to decide **which node has enough resources** to run the Pod.

Example:

```yaml
requests:
  cpu: 100m
  memory: 128Mi
```

- CPU Request = 100m = 0.1 CPU Core
- Memory Request = 128Mi

---

## What are Resource Limits?

Limits define the **maximum resources** a container is allowed to consume.

If a container exceeds:

- CPU Limit → CPU is throttled
- Memory Limit → Container is terminated (OOMKilled)

Example:

```yaml
limits:
  cpu: 250m
  memory: 256Mi
```

---

# Requests vs Limits

| Requests | Limits |
|----------|--------|
| Used by Scheduler | Enforced by Kubelet |
| Minimum Guaranteed Resources | Maximum Allowed Resources |
| Determines Pod Placement | Prevents Resource Abuse |

---

# Kubernetes Quality of Service (QoS)

Kubernetes assigns every Pod a QoS Class.

## Guaranteed

Requests = Limits for every resource.

Example:

```yaml
requests:
  cpu: 250m
  memory: 256Mi

limits:
  cpu: 250m
  memory: 256Mi
```

Highest priority during node pressure.

---

## Burstable

Requests are lower than Limits.

Example:

```yaml
requests:
  cpu: 100m
  memory: 128Mi

limits:
  cpu: 250m
  memory: 256Mi
```

Most production applications use this class.

---

## BestEffort

No requests or limits defined.

Lowest priority and the first Pods evicted during resource pressure.

---

# Task 1 – Resource Requests and Limits

## Manifest

**File:** `resource-pod.yaml`


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
  labels:
    app: resource-demo
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
```

---

## Apply

```bash
kubectl apply -f task1-resource-pod.yaml
```

---

## Verify

```bash
kubectl describe pod resource-demo
```

Expected:

- Requests
- Limits
- QoS Class

Observed QoS Class:

**Burstable**

Reason:

Requests and Limits are different.

---

## Screenshot

<img width="2152" height="1420" alt="image" src="https://github.com/user-attachments/assets/1c4fbe91-fb03-4e00-9f0a-672dc545de16" />


---

# Task 2 – OOMKilled (Memory Limit Exceeded)

A container exceeding its memory limit is immediately terminated by the Linux Kernel.

Unlike CPU, memory cannot be throttled.

---

## Manifest

**File:** `oom-pod.yaml`


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: oom-demo
spec:
  containers:
  - name: stress
    image: polinux/stress
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "200M", "--vm-hang", "1"]
    resources:
      limits:
        memory: 100Mi
      requests:
        memory: 50Mi
```

---

## Apply

```bash
kubectl apply -f task2-oom-pod.yaml
```

---

## Watch Pod

```bash
kubectl get pod oom-demo -w
```

---

## Verify

```bash
kubectl describe pod oom-demo
```

Expected:

- Reason: OOMKilled
- Exit Code: 137

Explanation:

```
128 + SIGKILL(9) = 137
```

---

## Key Learning

CPU overuse → Throttled

Memory overuse → Container Killed

---

## Screenshot

<img width="1692" height="388" alt="image" src="https://github.com/user-attachments/assets/eb9e202f-d7fd-468a-a4cd-92a16c7711ce" />

---

# Task 3 – Pending Pod (Insufficient Resources)

If requested resources exceed cluster capacity, Kubernetes cannot schedule the Pod.

The Pod remains in:

```
Pending
```

---

## Manifest

**File:** `pending-pod.yaml`


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pending-demo
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    resources:
      requests:
        cpu: "100"
        memory: 128Gi
      limits:
        cpu: "100"
        memory: 128Gi
```

---

## Apply

```bash
kubectl apply -f pending-pod.yaml
```

---

## Verify

```bash
kubectl get pod pending-demo
```

```bash
kubectl describe pod pending-demo
```

Expected Event:

```
FailedScheduling
Insufficient memory
```

(or similar scheduler message)

---

## Why?

The Scheduler cannot find any node with:

- 100 CPU
- 128Gi Memory

---

## Screenshot

<img width="2628" height="1702" alt="image" src="https://github.com/user-attachments/assets/b53ea6c5-1e88-489a-bada-bea673c3c516" />


---

# Kubernetes Health Probes

Kubernetes continuously checks container health using Probes.

There are three probe types:

| Probe | Purpose |
|--------|----------|
| Liveness | Restart unhealthy containers |
| Readiness | Control traffic to Pods |
| Startup | Give slow applications time to start |

---

# Task 4 – Liveness Probe

Liveness Probe detects applications that are stuck.

If the probe fails repeatedly, Kubernetes restarts the container automatically.

---

## Manifest

**File:** `liveness-pod.yaml`


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-demo
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command:
    - /bin/sh
    - -c
    - |
      touch /tmp/healthy
      sleep 30
      rm /tmp/healthy
      tail -f /dev/null
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      periodSeconds: 5
      failureThreshold: 3
```

---

## Apply

```bash
kubectl apply -f liveness-pod.yaml
```

---

## Watch

```bash
kubectl get pod liveness-demo -w
```

---

## Check Restart Count

```bash
kubectl get pod liveness-demo \
-o jsonpath='{.status.containerStatuses[0].restartCount}'
```

Expected:

Restart Count increases after `/tmp/healthy` is deleted.

---

## Screenshot

<img width="1478" height="336" alt="image" src="https://github.com/user-attachments/assets/26375e8d-ebc2-4db2-92a9-fc086924961a" />
<img width="2618" height="478" alt="image" src="https://github.com/user-attachments/assets/60287b42-0817-4ca1-9576-742db4675dfb" />
<img width="2102" height="154" alt="image" src="https://github.com/user-attachments/assets/e917c1f0-5e8a-4f76-9876-9a39e6452e00" />


---

# Task 5 – Readiness Probe

Readiness Probes determine whether a Pod is ready to receive traffic.

Unlike Liveness, failing Readiness does **NOT** restart the container.

Instead:

- Pod remains Running
- Removed from Service Endpoints

---

## Manifest

**File:** `readiness-pod.yaml`


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-demo
  labels:
    app: readiness-demo
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```

---

## Apply

```bash
kubectl apply -f readiness-pod.yaml
```

---

## Create Service

```bash
kubectl expose pod readiness-demo \
--port=80 \
--name=readiness-svc
```

---

## Verify Endpoints

```bash
kubectl get endpoints readiness-svc
```

---

## Break Readiness

```bash
kubectl exec readiness-demo -- \
rm /usr/share/nginx/html/index.html
```

---

## Observe

```bash
kubectl get pod readiness-demo -w
```

Expected:

```
READY

0/1
```

Container:

Still Running

Service Endpoints:

Empty

---

## Screenshot

<img width="2006" height="514" alt="image" src="https://github.com/user-attachments/assets/b423fdc7-34a5-4b1e-ae5d-ddc57728bf8d" />

<img width="1906" height="450" alt="image" src="https://github.com/user-attachments/assets/0008cba2-e037-4606-bfac-c7b01e0b825e" />


---

# Task 6 – Startup Probe

Some applications require extra startup time.

Without Startup Probes, Liveness may kill them before they finish initializing.

Startup Probe temporarily disables:

- Liveness Probe
- Readiness Probe

until startup succeeds.

---

## Manifest

**File:** `startup-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startup-demo
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command:
    - /bin/sh
    - -c
    - |
      sleep 20
      touch /tmp/started
      tail -f /dev/null
    startupProbe:
      exec:
        command:
        - cat
        - /tmp/started
      periodSeconds: 5
      failureThreshold: 12
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/started
      periodSeconds: 10
      initialDelaySeconds: 0
```

---

## Apply

```bash
kubectl apply -f startup-pod.yaml
```

---

## Watch

```bash
kubectl get pod startup-demo -w
```

---

## Observation

Container sleeps for:

```
20 seconds
```

Startup Probe waits until:

```
/tmp/started
```

exists.

Only after Startup succeeds does the Liveness Probe begin.

---

## What if failureThreshold = 2?

Probe Timing:

```
2 × 5 seconds = 10 seconds
```

Application Startup Time:

```
20 seconds
```

Result:

- Startup Probe fails
- Kubernetes kills the container
- Pod enters CrashLoopBackOff

---

## Screenshot

<img width="1426" height="346" alt="image" src="https://github.com/user-attachments/assets/fdcac38e-21f9-4548-b067-a662992a5dc0" />


---

# Task 7 – Cleanup

Delete all created resources.

```bash
kubectl delete pod \
resource-demo \
oom-demo \
pending-demo \
liveness-demo \
readiness-demo \
startup-demo
```

```bash
kubectl delete svc readiness-svc
```

---

# Key Takeaways

- Requests help the Scheduler place Pods.
- Limits prevent containers from consuming excessive resources.
- CPU overuse results in throttling, while memory overuse leads to OOMKilled.
- QoS Classes determine how Kubernetes prioritizes Pods under resource pressure.
- Liveness Probes restart unhealthy containers.
- Readiness Probes remove unhealthy Pods from Service endpoints without restarting them.
- Startup Probes provide additional startup time for slow applications and protect them from premature restarts.

---

# Commands Used

```bash
kubectl apply -f <file>

kubectl describe pod <pod-name>

kubectl get pod -w

kubectl get endpoints readiness-svc

kubectl expose pod readiness-demo --port=80 --name=readiness-svc

kubectl exec readiness-demo -- rm /usr/share/nginx/html/index.html

kubectl delete pod resource-demo oom-demo pending-demo liveness-demo readiness-demo startup-demo

kubectl delete svc readiness-svc
```

---

# Files Created

```
resource-pod.yaml

oom-pod.yaml

pending-pod.yaml

liveness-pod.yaml

readiness-pod.yaml

startup-pod.yaml

day-57-resources-probes.md
```

---
