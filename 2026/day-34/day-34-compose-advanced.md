# Day 34 – Docker Compose: Real-World Multi-Container Apps

## Objective

The goal of this exercise was to build a production-like multi-container application using Docker Compose.

The stack consists of:

* Node.js Web Application
* PostgreSQL Database
* Redis Cache

I also explored:

* Service dependencies (`depends_on`)
* Healthchecks
* Restart policies
* Custom Dockerfile builds
* Named networks and volumes
* Container scaling

---

# Architecture

```text
            +-------------+
            |   Browser   |
            +------+------+
                   |
                   v
            +-------------+
            |  Node.js App |
            +------+------+
                   |
        +----------+----------+
        |                     |
        v                     v
+---------------+     +---------------+
| PostgreSQL DB |     | Redis Cache   |
+---------------+     +---------------+
```

---

# Task 1 – Create a 3-Service Stack

I created a Docker Compose configuration containing:

1. Node.js Web Application
2. PostgreSQL Database
3. Redis Cache

The web application was built using my existing Dockerfile from the Node.js Getting Started project.

## Docker Compose File

```yaml
services:
  db:
    image: postgres:16
    container_name: postgres-db

    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin123
      POSTGRES_DB: mydb

    volumes:
      - postgres_data:/var/lib/postgresql/data

    restart: always

    healthcheck:
      test: ["CMD", "pg_isready", "-U", "admin", "-d", "mydb"]
      interval: 10s
      timeout: 5s
      retries: 5

    networks:
      - backend

  redis:
    image: redis:7

    container_name: redis-cache

    networks:
      - backend

  web:
    build: ./app

    container_name: node-web

    ports:
      - "5006:5006"

    depends_on:
      db:
        condition: service_healthy

    networks:
      - backend

networks:
  backend:

volumes:
  postgres_data:
```
`docker compose up -d`

<img width="735" height="121" alt="image" src="https://github.com/user-attachments/assets/cad1694b-e365-4d2c-ae13-e1d4a7d4310c" />

`docker compose ps`

<img width="1223" height="108" alt="image" src="https://github.com/user-attachments/assets/c51a5f4b-c838-41c3-94ba-8ba16c27d327" />


`<your ec2-ip> : 5006`

<img width="1676" height="820" alt="image" src="https://github.com/user-attachments/assets/4b3b6a48-1032-45eb-8de5-d202b7547857" />


---

# Task 2 – Depends On and Healthchecks

The Node.js application depends on PostgreSQL.

Using:

```yaml
depends_on:
  db:
    condition: service_healthy
```

ensures that the application starts only after PostgreSQL passes its healthcheck.

Healthcheck used:

```yaml
healthcheck:
  test: ["CMD", "pg_isready", "-U", "admin", "-d", "mydb"]
  interval: 10s
  timeout: 5s
  retries: 5
```

### Commands Used

```bash
docker compose up -d
docker compose ps
docker compose logs -f
```

### Observation

Without a healthcheck, the application may start before the database is ready.

Using `service_healthy` prevents startup failures caused by database initialization delays.

### Screenshot

Add screenshot:

`docker compose logs -f`

<img width="1225" height="515" alt="image" src="https://github.com/user-attachments/assets/c57d8b4d-0e3e-4c38-be28-f27f09728873" />

---

# Task 3 – Restart Policies

I configured PostgreSQL with:

```yaml
restart: always
```

### Test Performed

```bash
docker kill postgres-db
```

Docker automatically restarted the database container.

### Result

The container came back online without manual intervention.

### Restart Policies Comparison

| Policy         | Behavior                                        |
| -------------- | ----------------------------------------------- |
| no             | Never restart                                   |
| always         | Always restart                                  |
| on-failure     | Restart only when container exits with an error |
| unless-stopped | Restart unless manually stopped                 |

### When to Use

#### restart: always

* Databases
* Critical production services
* Monitoring systems

#### restart: on-failure

* Batch jobs
* Scheduled scripts
* Worker containers

### Screenshot

Add screenshot:

```bash
docker exec -it postgres-db pkill -9 postgres
docker ps
```

<img width="1214" height="324" alt="image" src="https://github.com/user-attachments/assets/b80f91fc-b9b9-4a6d-b6b9-bbb6a5e3169c" />

---

# Task 4 – Custom Dockerfile Build

The web application image was built from a local Dockerfile using:

```yaml
web:
  build: .
```

After modifying the application code, I rebuilt the stack using:

```bash
docker compose up --build -d
```

### Observation

Docker rebuilt the application image and recreated the web container automatically.

### Screenshot

Add screenshot:

```bash
docker compose up --build -d
```

---

# Task 5 – Named Networks and Volumes

## Network

```yaml
networks:
  backend:
```

## Volume

```yaml
volumes:
  postgres_data:
```

### Verify Networks

```bash
docker network ls
```

### Verify Volumes

```bash
docker volume ls
```

### Observation

Using named volumes ensures PostgreSQL data persists even if containers are removed.

Named networks provide isolated communication between application services.

### Screenshots

Add screenshots:

```bash
docker network ls
```

<img width="680" height="200" alt="image" src="https://github.com/user-attachments/assets/56f25c0e-9c50-4b16-8c97-1d2527b9a1fe" />


```bash
docker volume ls
```

<img width="639" height="137" alt="image" src="https://github.com/user-attachments/assets/22cedda7-a32e-41a5-9e1f-89f837eeee7a" />

---

# Task 6 – Scaling (Bonus)

I attempted to scale the web service:

```bash
docker compose up --scale web=3 -d
```

### Observation

Scaling caused a port conflict because all replicas attempted to bind to:

```yaml
ports:
  - "5006:5006"
```

Only one container can use a host port at a time.

### Why Scaling Fails

```text
web-1 --> port 5006
web-2 --> port 5006
web-3 --> port 5006
```

Docker cannot assign the same host port to multiple containers.

### Real-World Solution

Production environments typically use:

* Nginx
* HAProxy
* Traefik
* Kubernetes Services

These tools distribute traffic among multiple replicas.

### Screenshot

Add screenshot:

```bash
docker compose up --scale web=3 -d
```

<img width="1212" height="134" alt="image" src="https://github.com/user-attachments/assets/07b50e6d-aea3-4b2d-87ac-108a739f091a" />

---

# Commands Used

```bash
docker compose up -d

docker compose up --build -d

docker compose down

docker compose ps

docker compose logs -f

docker network ls

docker volume ls

docker exec -it postgres-db pkill -9 postgres

docker compose up --scale web=3 -d
```

---

# Key Learnings

* Docker Compose simplifies multi-container deployments.
* Healthchecks ensure services are truly ready before dependent services start.
* Restart policies improve application resilience.
* Named volumes preserve data across container recreation.
* Named networks improve service communication.
* Scaling containers requires a load balancer and cannot rely on a single mapped port.
* Docker Compose is an effective tool for managing production-like application stacks.
