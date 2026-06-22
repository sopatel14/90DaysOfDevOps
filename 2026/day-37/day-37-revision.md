# Day 37 – Docker Revision & Self Assessment

## Self-Assessment Checklist

| Topic                                    | Status   |
| ---------------------------------------- | -------- |
| Run a container from Docker Hub          | ✅ Can Do |
| List, stop, remove containers and images | ✅ Can Do |
| Explain image layers and caching         | ✅ Can Do |
| Write a Dockerfile from scratch          | ✅ Can Do |
| Explain CMD vs ENTRYPOINT                | ✅ Can Do |
| Build and tag custom images              | ✅ Can Do |
| Create and use named volumes             | ✅ Can Do |
| Use bind mounts                          | ⚠️ Shaky |
| Create custom networks                   | ✅ Can Do |
| Write docker-compose.yml                 | ✅ Can Do |
| Use environment variables in Compose     | ✅ Can Do |
| Write multi-stage Dockerfiles            | ✅ Can Do |
| Push images to Docker Hub                | ✅ Can Do |
| Use healthchecks and depends_on          | ✅ Can Do |

---

## Quick-Fire Questions

### 1. What is the difference between an image and a container?

An image is a read-only template containing application code, dependencies, and configuration. A container is a running instance of that image.

---

### 2. What happens to data inside a container when you remove it?

Container data is lost when the container is removed unless the data is stored in a Docker volume or bind mount.

---

### 3. How do two containers on the same custom network communicate?

Containers communicate using their service name or container name through Docker's built-in DNS.

Example:

```text
postgres-db:5432
redis-cache:6379
```

---

### 4. What does `docker compose down -v` do differently from `docker compose down`?

`docker compose down`

* Stops and removes containers
* Preserves volumes

`docker compose down -v`

* Stops and removes containers
* Removes associated volumes

---

### 5. Why are multi-stage builds useful?

Multi-stage builds reduce image size by keeping only the files needed for runtime and removing build dependencies from the final image.

Benefits:

* Smaller images
* Faster deployments
* Improved security

---

### 6. What is the difference between COPY and ADD?

COPY:

* Copies files/directories into the image

ADD:

* Copies files
* Can extract local tar archives
* Supports remote URLs

COPY is generally preferred.

---

### 7. What does `-p 8080:80` mean?

Maps:

```text
Host Port      → 8080
Container Port → 80
```

Requests sent to port 8080 on the host are forwarded to port 80 inside the container.

---

### 8. How do you check Docker disk usage?

```bash
docker system df
```

Provides information about:

* Images
* Containers
* Volumes
* Build cache

---

## Weak Areas Reviewed

### Topic 1: Bind Mounts

Revisited bind mounts by mounting a local directory into a container.

Example:

```bash
docker run -v $(pwd):/app nginx
```

Learning:

* Changes on host are immediately reflected inside the container.
* Useful for development environments.

---

### Topic 2: Docker Networking

Revisited custom networks.

Commands used:

```bash
docker network create backend
docker network ls
docker network inspect backend
```

Learning:

* Containers on the same network can communicate using container names.
* No need to expose internal ports for inter-container communication.

---

## Key Takeaways From Days 29–36

* Docker images are built in layers.
* Containers are isolated runtime environments.
* Volumes provide persistent storage.
* Networks enable container communication.
* Docker Compose simplifies multi-container deployments.
* Multi-stage builds create smaller production images.
* Docker Hub enables image sharing and deployment.
* Health checks improve service reliability.
* Environment variables help separate configuration from code.

---

## Revision Summary

This revision day helped reinforce Docker fundamentals, Docker Compose workflows, image management, networking, storage, and deployment best practices. I feel confident building and deploying containerized applications using Docker and Docker Compose.
