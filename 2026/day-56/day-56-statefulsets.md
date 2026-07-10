# Day 56 – Kubernetes StatefulSets

## Overview

Today I learned about **StatefulSets**, the Kubernetes workload designed for applications that require **stable identities**, **persistent storage**, and **ordered deployment**.

Unlike Deployments, StatefulSets are mainly used for **stateful applications** such as:

- MySQL
- PostgreSQL
- MongoDB
- Kafka
- Elasticsearch
- Redis (cluster mode)

---

# Why StatefulSets?

Deployments are perfect for stateless applications where Pods can be replaced at any time without affecting the application.

However, databases and distributed applications require:

- Stable pod names
- Persistent storage for each replica
- Predictable startup order
- Stable DNS names

StatefulSets provide all these features.

---

# Deployment vs StatefulSet

| Feature | Deployment | StatefulSet |
|----------|------------|-------------|
| Pod Names | Random | Stable (app-0, app-1, app-2) |
| Startup Order | Parallel | Ordered |
| Shutdown Order | Random | Reverse Order |
| Storage | Shared PVC (optional) | Dedicated PVC per Pod |
| Network Identity | Random | Stable DNS |
| Best For | Stateless Applications | Stateful Applications |

---

# Task 1 – Understanding the Problem

First, I created an nginx Deployment with three replicas.

```bash
kubectl create deployment nginx-deployment --image=nginx --replicas=3
```

Check Pods

```bash
kubectl get pods
```

Example Output

```
nginx-deployment-6d4cf56d7b-lm5mk
nginx-deployment-6d4cf56d7b-v2t7m
nginx-deployment-6d4cf56d7b-xkrmt
```

Delete one Pod

```bash
kubectl delete pod nginx-deployment-6d4cf56d7b-lm5mk
```

A new Pod was created with a completely different name.

This behavior is acceptable for web servers but causes problems for databases because:

- Cluster members lose their identity.
- Configuration files cannot rely on stable hostnames.
- Replication becomes difficult.
- Peer discovery becomes unreliable.

Delete the Deployment

```bash
kubectl delete deployment nginx-deployment
```

---

## Screenshot

<img width="1850" height="554" alt="image" src="https://github.com/user-attachments/assets/6721920f-cd13-4ffc-8ef9-55f0d58892e9" />


---

# Task 2 – Create a Headless Service

Created a Headless Service.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None
  selector:
    app: nginx-stateful
  ports:
    - port: 80
      targetPort: 80
```

Apply

```bash
kubectl apply -f headless-service.yaml
```

Verify

```bash
kubectl get svc
```
## Screenshot

<img width="1580" height="332" alt="image" src="https://github.com/user-attachments/assets/eebc2ca5-e84e-4409-8031-ede1edb36458" />



The **CLUSTER-IP** shows **None**, confirming it is a Headless Service.

---

## Why Headless Service?

Instead of load balancing traffic, Kubernetes creates DNS entries for every Pod.

DNS format:

```
<pod-name>.<service-name>.<namespace>.svc.cluster.local
```

Example

```
nginx-stateful-0.nginx-headless.default.svc.cluster.local
```

---

## Screenshot

📸 **Insert Screenshot**

- `kubectl get svc`

---

# Task 3 – Create a StatefulSet

Created the StatefulSet.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-stateful
spec:
  serviceName: nginx-headless
  replicas: 3
  selector:
    matchLabels:
      app: nginx-stateful
  template:
    metadata:
      labels:
        app: nginx-stateful
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: web-data
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: web-data
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 100Mi
```

Apply

```bash
kubectl apply -f statefulset.yaml
```

Watch Pods

```bash
kubectl get pods -l app=nginx-stateful -w
```

Pods were created in order:

```
nginx-stateful-0
nginx-stateful-1
nginx-stateful-2
```

Check PVCs

```bash
kubectl get pvc
```

PVCs

```
web-data-nginx-stateful-0
web-data-nginx-stateful-1
web-data-nginx-stateful-2
```

Each Pod automatically received its own Persistent Volume Claim.

---

## Screenshot

<img width="1600" height="494" alt="image" src="https://github.com/user-attachments/assets/3a2a783f-30fd-461d-b9a7-cf17f732b8df" />


<img width="2242" height="292" alt="image" src="https://github.com/user-attachments/assets/eb25ced0-1947-4e0f-a445-0bf7348225fb" />

---

# Task 4 – Stable Network Identity

Started a temporary BusyBox Pod.

```bash
kubectl run -it --rm busybox --image=busybox --restart=Never -- sh
```

Inside BusyBox

```bash
nslookup nginx-stateful-0.nginx-headless.default.svc.cluster.local

nslookup nginx-stateful-1.nginx-headless.default.svc.cluster.local

nslookup nginx-stateful-2.nginx-headless.default.svc.cluster.local
```

Check Pod IPs

```bash
kubectl get pods -o wide
```

The DNS records resolved directly to each Pod's IP.

This gives every Pod a stable network identity.

---

## Screenshot

<img width="2328" height="1104" alt="image" src="https://github.com/user-attachments/assets/dceb6356-edd2-4971-aa3a-275a3b6122e9" />


<img width="1914" height="240" alt="image" src="https://github.com/user-attachments/assets/6f139460-e35f-4d79-a2d4-ebc0797f80fd" />


---

# Task 5 – Persistent Storage

Write unique content into each Pod.

```bash
kubectl exec nginx-stateful-0 -- sh -c "echo 'Data from web-0' > /usr/share/nginx/html/index.html"

kubectl exec nginx-stateful-1 -- sh -c "echo 'Data from web-1' > /usr/share/nginx/html/index.html"

kubectl exec nginx-stateful-2 -- sh -c "echo 'Data from web-2' > /usr/share/nginx/html/index.html"
```

Verify

```bash
kubectl exec nginx-stateful-0 -- cat /usr/share/nginx/html/index.html
```

Delete Pod

```bash
kubectl delete pod nginx-stateful-0
```

Wait until it becomes Running again.

Verify

```bash
kubectl exec nginx-stateful-0 -- cat /usr/share/nginx/html/index.html
```

Result

The original data remained intact because the Pod reused the same PVC.

---

## Screenshot


<img width="2348" height="570" alt="image" src="https://github.com/user-attachments/assets/b6b53514-cfc7-42d1-a279-3476b010b462" />

<img width="1882" height="506" alt="image" src="https://github.com/user-attachments/assets/b9fa63ed-6d2b-4446-be50-89bd309a7259" />


---

# Task 6 – Ordered Scaling

Scale Up

```bash
kubectl scale statefulset nginx-stateful --replicas=5
```

Pods were created in order

```
nginx-stateful-3

nginx-stateful-4
```

Scale Down

```bash
kubectl scale statefulset nginx-stateful --replicas=3
```

Pods terminated in reverse order

```
nginx-stateful-4

nginx-stateful-3
```

Check PVCs

```bash
kubectl get pvc
```

Even after scaling down, all five PVCs still existed.

This protects application data if replicas are added again later.

---

## Screenshot

<img width="1774" height="574" alt="image" src="https://github.com/user-attachments/assets/cec63662-2aa8-491a-8d96-3bcaa30d77ba" />

<img width="1175" height="353" alt="image" src="https://github.com/user-attachments/assets/9be82edf-f24f-4eb2-a082-b6219fe1d631" />



---

# Task 7 – Cleanup

Delete StatefulSet

```bash
kubectl delete statefulset nginx-stateful
```

Delete Headless Service

```bash
kubectl delete svc nginx-headless
```

PVCs still existed.

```bash
kubectl get pvc
```

Delete PVCs manually.

```bash
kubectl delete pvc --all
```

This behavior prevents accidental data loss.

---

## Screenshot

<img width="2328" height="546" alt="image" src="https://github.com/user-attachments/assets/d080f77c-d428-44a7-a3fc-46093814eeb9" />


---

# Key Learnings

- StatefulSets are designed for stateful applications.
- Pods receive predictable names.
- Pods start sequentially.
- Pods terminate in reverse order.
- Every replica receives its own Persistent Volume Claim.
- Headless Services provide stable DNS names.
- Storage survives Pod recreation.
- PVCs are retained even after scaling down or deleting the StatefulSet.

---

# Commands Used

```bash
kubectl create deployment nginx-deployment --image=nginx --replicas=3

kubectl get pods

kubectl delete deployment nginx-deployment

kubectl apply -f headless-service.yaml

kubectl get svc

kubectl apply -f statefulset.yaml

kubectl get pods -w

kubectl get pvc

kubectl run -it --rm busybox --image=busybox --restart=Never -- sh

nslookup nginx-stateful-0.nginx-headless.default.svc.cluster.local

kubectl get pods -o wide

kubectl exec nginx-stateful-0 -- sh -c "echo 'Data from web-0' > /usr/share/nginx/html/index.html"

kubectl delete pod nginx-stateful-0

kubectl scale statefulset nginx-stateful --replicas=5

kubectl scale statefulset nginx-stateful --replicas=3

kubectl delete statefulset nginx-stateful

kubectl delete svc nginx-headless

kubectl delete pvc --all
```

---

# Conclusion

StatefulSets solve the challenges of running stateful workloads in Kubernetes by providing stable identities, persistent storage, predictable scaling, and reliable networking. They are the preferred workload for databases, distributed systems, and clustered applications where maintaining state is critical.
