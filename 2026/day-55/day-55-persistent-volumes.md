# Day 55 – Persistent Volumes (PV) & Persistent Volume Claims (PVC)

## 📌 Objective

Learn how Kubernetes provides persistent storage using **Persistent Volumes (PV)** and **Persistent Volume Claims (PVC)** so that application data survives Pod restarts and deletion.

---

# 📖 Overview

Containers are **ephemeral**, meaning their filesystem disappears when the Pod is deleted.

This creates problems for:

- Databases
- Application logs
- User uploads
- Configuration data
- Any application requiring persistent storage

Kubernetes solves this using:

- **Persistent Volume (PV)** – Actual storage resource in the cluster.
- **Persistent Volume Claim (PVC)** – Storage request made by applications.

---

# PV vs PVC

| Persistent Volume (PV) | Persistent Volume Claim (PVC) |
|-------------------------|-------------------------------|
| Cluster-wide resource | Namespace resource |
| Created by Admin / StorageClass | Created by Developer |
| Represents actual storage | Requests storage |
| Can exist without Pods | Used by Pods |

### Binding Process

```text
PVC Request
      │
      ▼
Kubernetes Finds Matching PV
      │
      ▼
PV <-------> PVC
      │
      ▼
Pod Uses PVC
```

---

# Task 1 – Demonstrate Ephemeral Storage

## Step 1 – Create Pod

**File:** `ephemeral-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ephemeral-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo $(date) >> /data/message.txt; sleep 10; done"]
    volumeMounts:
    - name: temp-storage
      mountPath: /data
  volumes:
  - name: temp-storage
    emptyDir: {}
```

## Step 2 – Apply

```bash
kubectl apply -f ephemeral-pod.yaml

kubectl exec ephemeral-pod -- cat /data/message.txt
```

## Step 3 – Delete & Recreate

```bash
kubectl delete pod ephemeral-pod

kubectl apply -f ephemeral-pod.yaml

kubectl exec ephemeral-pod -- cat /data/message.txt
```

### ✅ Observation

Data is lost because `emptyDir` exists only while the Pod exists.

---

## 📸 Screenshot

<img width="1676" height="328" alt="image" src="https://github.com/user-attachments/assets/65d32613-53fd-4b21-b77c-5b09bf6cf6f1" />

<img width="1782" height="274" alt="image" src="https://github.com/user-attachments/assets/649d722b-6e09-4902-b95d-3795d014dd7f" />



---

# Task 2 – Create Persistent Volume

## File: `persistent-volume.yaml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /tmp/k8s-pv-data
```

## Apply

```bash
kubectl apply -f persistent-volume.yaml

kubectl get pv
```

Expected Status:

```text
STATUS: Available
```

---

## Access Modes

| Mode | Meaning |
|------|---------|
| ReadWriteOnce (RWO) | Read/Write by one node |
| ReadOnlyMany (ROX) | Read-only by many nodes |
| ReadWriteMany (RWX) | Read/Write by many nodes |

> **Note:** `hostPath` is only for learning purposes.

---

## 📸 Screenshot 

<img width="2250" height="372" alt="image" src="https://github.com/user-attachments/assets/e6eb73c1-808f-42cc-acd5-58d09fe4f0da" />


---

# Task 3 – Create Persistent Volume Claim

## File: `persistent-volume-claim.yaml`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

## Apply

```bash
kubectl apply -f persistent-volume-claim.yaml

kubectl get pvc

kubectl get pv
```

Expected Output

```text
PVC STATUS : Bound

PV STATUS : Bound
```

Example

```text
kubectl get pvc

NAME     STATUS   VOLUME   CAPACITY
my-pvc   Bound    my-pv    1Gi
```

---

## 📸 Screenshot

<img width="2262" height="432" alt="image" src="https://github.com/user-attachments/assets/d84efbe8-d3c8-4b42-82b7-8135989830c4" />


---

# Task 4 – Use PVC in a Pod

## File: `pod-with-pvc.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: persistent-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo $(date) >> /data/message.txt; sleep 10; done"]
    volumeMounts:
    - name: persistent-storage
      mountPath: /data
  volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: my-pvc
```

## Test Persistence

```bash
kubectl apply -f pod-with-pvc.yaml

kubectl exec persistent-pod -- cat /data/message.txt

kubectl delete pod persistent-pod

kubectl apply -f pod-with-pvc.yaml

kubectl exec persistent-pod -- cat /data/message.txt
```

### ✅ Observation

Old data still exists after Pod recreation because it is stored inside the Persistent Volume.

---

## 📸 Screenshot

<img width="1848" height="762" alt="image" src="https://github.com/user-attachments/assets/4d0cb8b8-8efe-4ea0-8948-3002f6a1c137" />

---

# Task 5 – Explore StorageClass

List Storage Classes

```bash
kubectl get storageclass

kubectl describe storageclass standard
```

Important Fields

- Provisioner
- Reclaim Policy
- Volume Binding Mode

---

## 📸 Screenshot

<img width="2256" height="664" alt="image" src="https://github.com/user-attachments/assets/b7d920f2-5923-43ba-96e8-b565b79948a8" />

---

# Task 6 – Dynamic Provisioning

## Create Dynamic PVC

**File:** `dynamic-pvc.yaml`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: standard
```

---


Apply

```bash
kubectl apply -f dynamic-pvc.yaml

kubectl get pvc

kubectl get pv
```

---

## Create Pod

**File:** `dynamic-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dynamic-pod
spec:
  containers:
  - name: data-writer
    image: alpine
    command:
      - "/bin/sh"
      - "-c"
      - "echo 'Dynamic Data Success' > /data/test.txt && sleep 3600"
    volumeMounts:
    - name: storage-volume
      mountPath: /data
  volumes:
  - name: storage-volume
    persistentVolumeClaim:
      claimName: dynamic-pvc
```

Test

```bash
kubectl apply -f dynamic-pod.yaml

kubectl exec dynamic-pod -- cat /data/test.txt

kubectl delete pod dynamic-pod

kubectl apply -f dynamic-pod.yaml

kubectl exec dynamic-pod -- cat /data/test.txt
```

List PVs

```bash
kubectl get pv
```

---

## 📸 Screenshot 

<img width="2250" height="572" alt="image" src="https://github.com/user-attachments/assets/6e658de8-7f79-434e-95a9-388c6d1fde24" />

<img width="2260" height="346" alt="image" src="https://github.com/user-attachments/assets/590422b0-ea62-4678-b3ff-3eda03d432a7" />


---

# Task 7 – Cleanup

Delete Pods

```bash
kubectl delete pod --all
```

Delete PVCs

```bash
kubectl delete pvc my-pvc

kubectl delete pvc dynamic-pvc
```

Check PV Status

```bash
kubectl get pv
```

Delete Remaining PV

```bash
kubectl delete pv my-pv

kubectl get pv
```
---

# Persistent Volume Lifecycle

```text
Available
    │
    ▼
Bound
    │
    ▼
Released
    │
    ▼
Available (after cleanup)
```

---

# Reclaim Policies

| Policy | Behaviour |
|---------|-----------|
| Retain | Keeps storage after PVC deletion |
| Delete | Deletes storage automatically |

---

# Access Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| RWO | ReadWriteOnce | Most applications |
| ROX | ReadOnlyMany | Shared configuration |
| RWX | ReadWriteMany | Shared storage |

---

# Useful Commands

```bash
# View storage resources
kubectl get pv,pvc,sc

# Describe PV
kubectl describe pv my-pv

# Describe PVC
kubectl describe pvc my-pvc

# Describe Pod
kubectl describe pod persistent-pod

# Cleanup
kubectl delete pod --all
kubectl delete pvc --all
kubectl delete pv --all
```

---

# Troubleshooting

## PVC Pending

```bash
kubectl describe pvc dynamic-pvc
```

Possible causes:

- No matching PV
- Wrong StorageClass
- Insufficient capacity
- Access mode mismatch

---

## Fix Default StorageClass

```bash
kubectl patch storageclass <name> -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

---

# Key Learnings

- `emptyDir` is temporary storage.
- Persistent Volumes survive Pod deletion.
- PVC requests storage from Kubernetes.
- PV and PVC bind together automatically.
- StorageClass enables dynamic provisioning.
- Reclaim Policies decide whether data is retained or deleted.
- Static provisioning requires manual PV creation.
- Dynamic provisioning automatically creates storage when a PVC is requested.

---

# Files Created

```text
ephemeral-pod.yaml
persistent-volume.yaml
persistent-volume-claim.yaml
pod-with-pvc.yaml
dynamic-pvc.yaml
pv-for-dynamic.yaml
dynamic-pod.yaml
```

---

# Final Outcome

✔ Understood ephemeral vs persistent storage.

✔ Created a Persistent Volume (PV).

✔ Bound a Persistent Volume Claim (PVC).

✔ Mounted PVC inside Pods.

✔ Verified data survives Pod recreation.

✔ Explored StorageClasses.

✔ Practiced dynamic provisioning.

✔ Learned reclaim policies and access modes.

---
