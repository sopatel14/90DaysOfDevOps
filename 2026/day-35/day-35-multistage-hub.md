# Day 35 – Multi-Stage Builds & Docker Hub

## 🚀 Objective

The objective of this project is to optimize Docker images using **Multi-Stage Builds** and publish them to **Docker Hub**. This approach helps reduce image size, improve security, and align with industry-standard DevOps practices.

---

## 📌 Project Overview

A simple Node.js application was containerized using two different approaches:

* Single-stage Docker build (Baseline)
* Multi-stage Docker build (Optimized)
* Docker Hub for image distribution and version management

---

# 🧱 Task 1: Single-Stage Build (Baseline Image)

## Dockerfile

```dockerfile
FROM node:22

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

## Build Command

```bash
docker build -t node-app-single .
```

## Image Size (Before Optimization)

```text
node-app-single: ~800MB – 1.2GB
```

### 🧠 Observation

The single-stage image is significantly larger because it contains:

* Build tools
* Development dependencies
* Package manager cache
* Source files not required at runtime
* Full Node.js environment

---

# ⚡ Task 2: Multi-Stage Build (Optimized Image)

## Dockerfile

```dockerfile
# Stage 1: Build Stage
FROM node:22 AS builder

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

# Stage 2: Production Stage
FROM node:18-alpine

WORKDIR /app

COPY --from=builder /app .

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

USER appuser

EXPOSE 5006

CMD ["npm", "start"]
```

## Build Command

```bash
docker build -f Docker-Multistage -t node-app-multi .
```

## Image Size (After Optimization)

```text
node-app-multi: ~100MB – 300MB
```

---

# 📊 Size Comparison

<img width="1138" height="284" alt="image" src="https://github.com/user-attachments/assets/b98c0364-56b1-471c-9bd7-4efd5125032a" />


---

## 🧠 Why Multi-Stage Builds Are Better

Multi-stage builds reduce image size because:

* Build dependencies are discarded after compilation.
* Only required runtime files are copied into the final image.
* Lightweight base images such as Alpine Linux are used.
* The final image contains fewer packages, reducing the attack surface.

### Benefits

✅ Smaller image size

✅ Faster image downloads and deployments

✅ Reduced storage consumption

✅ Improved security posture

✅ Better CI/CD performance

---

# 🔐 Task 3: Push Image to Docker Hub

## Step 1: Login to Docker Hub

```bash
docker login
```

## Step 2: Tag the Image

```bash
docker tag node-app-multi <dockerhub-username>/node-app-multi:1.0
```

## Step 3: Push the Image

```bash
docker push <dockerhub-username>/node-app-multi:1.0
```
<img width="1680" height="723" alt="image" src="https://github.com/user-attachments/assets/30b6967f-2bdc-4d7c-bf5d-aa518c477fef" />


---

## 📥 Verify the Published Image

Remove the local image:

```bash
docker rmi <dockerhub-username>/node-app-multi:1.0
```

Pull the image from Docker Hub:

```bash
docker pull <dockerhub-username>/node-app-multi:1.0
```

<img width="1130" height="766" alt="image" src="https://github.com/user-attachments/assets/fb7c422f-ae88-485e-8f55-05f9936d2b1d" />


Run the container:

```bash
docker run -p 3000:3000 <dockerhub-username>/node-app-multi:1.0
```

<img width="1576" height="645" alt="image" src="https://github.com/user-attachments/assets/49d93434-f347-4496-b179-b601faf35689" />


---

# 🐳 Task 4: Docker Hub Repository Management

After publishing the image:

### Repository Improvements

* Added repository description
* Added project documentation
* Managed image tags

### Versioning Strategy

| Tag    | Purpose                          |
| ------ | -------------------------------- |
| latest | Default version pulled by Docker |
| 1.0    | Stable production release        |

Example:

```bash
docker pull <dockerhub-username>/node-app-multi:latest
```

```bash
docker pull <dockerhub-username>/node-app-multi:1.0
```

---

# 🛡️ Task 5: Security & Best Practices

## Implemented Improvements

### 1. Multi-Stage Build

Reduces image size and removes unnecessary build dependencies.

### 2. Alpine-Based Runtime Image

```dockerfile
FROM node:22-alpine
```

Provides a lightweight production environment.

### 3. Non-Root User

```dockerfile
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

USER appuser
```

Prevents the application from running with root privileges.

---

## Additional Recommended Best Practices

### Use Specific Base Image Versions

Avoid:

```dockerfile
FROM node:latest
```

Prefer:

```dockerfile
FROM node:18-alpine
```

### Use `npm ci` in Production

Replace:

```dockerfile
RUN npm install
```

With:

```dockerfile
RUN npm ci --only=production
```

Benefits:

* Faster installs
* Deterministic builds
* Better CI/CD reliability

### Reduce Layers

Combine commands where possible to create fewer image layers.


---

# 📌 Key Learnings

* Multi-stage builds significantly reduce Docker image size.
* Smaller images improve deployment speed and security.
* Docker Hub enables efficient image distribution and collaboration.
* Proper image tagging simplifies version management.
* Running containers as non-root users enhances security.
* Alpine-based images help optimize production workloads.
* Following Docker best practices leads to more maintainable and scalable deployments.

---
