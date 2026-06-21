# Day 36 – Docker Project: Dockerize a Full Application

## Objective

Today's goal was to Dockerize a complete application and deploy it using Docker Compose. The project includes a Node.js application, PostgreSQL database, Redis cache, Docker networking, persistent volumes, health checks, and Docker Hub image distribution.

---

## Project Chosen

I used my existing Node.js application and extended it into a complete multi-container Docker deployment.

### Why This Project?

This project demonstrates several real-world Docker concepts:

* Application containerization
* Multi-container architecture
* Database integration
* Redis caching layer
* Docker Compose orchestration
* Persistent storage
* Health checks
* Environment variable management
* Docker Hub image deployment

---

## Project Architecture

```text
                    ┌───────────────┐
                    │    Browser    │
                    └───────┬───────┘
                            │
                            ▼
                  ┌─────────────────┐
                  │   Node.js App   │
                  │   (Container)   │
                  └───────┬─────────┘
                          │
          ┌───────────────┴───────────────┐
          ▼                               ▼
┌─────────────────┐             ┌─────────────────┐
│   PostgreSQL    │             │      Redis      │
│   Container     │             │   Container     │
└─────────────────┘             └─────────────────┘
```

All services communicate through a custom Docker network.

---

## Dockerfile

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 5006

CMD ["npm", "start"]
```

### Dockerfile Explanation

| Instruction         | Purpose                   |
| ------------------- | ------------------------- |
| FROM node:18-alpine | Lightweight Node.js image |
| WORKDIR /app        | Sets working directory    |
| COPY package*.json  | Copies dependency files   |
| RUN npm install     | Installs dependencies     |
| COPY . .            | Copies application code   |
| EXPOSE 5006         | Opens application port    |
| CMD                 | Starts application        |

---

## Docker Compose Configuration

```yaml
services:
  db:
    image: postgres:16
    container_name: postgres-db

    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}

    volumes:
      - postgres_data:/var/lib/postgresql/data

    restart: always

    healthcheck:
      test: ["CMD", "pg_isready", "-U", "${POSTGRES_USER}", "-d", "${POSTGRES_DB}"]
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
    image: sopatel264/node-app-multi:latest
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

---

## Environment Variables

### .env

```env
POSTGRES_USER=admin
POSTGRES_PASSWORD=admin123
POSTGRES_DB=mydb
```

### .env.example

```env
POSTGRES_USER=your_user
POSTGRES_PASSWORD=your_password
POSTGRES_DB=your_database
```

The actual `.env` file is excluded from Git using `.gitignore`.

---

## .gitignore

```gitignore
node_modules/
.env
*.log
```

---

## Building the Image

```bash
docker build -t node-app-multi .
```

Verify image:

```bash
docker images
```

---

## Starting the Application

```bash
docker compose up -d
```

Verify containers:

```bash
docker compose ps
```

```bash
docker ps
```

---

## Docker Volumes

Persistent storage is configured using Docker volumes.

```bash
docker volume ls
```

Volume:

```text
postgres_data
```

This ensures PostgreSQL data survives container restarts.

---

## Docker Networking

A custom bridge network is automatically created by Docker Compose.

```bash
docker network ls
```

Network:

```text
backend
```

This allows the containers to communicate securely.

---

## Health Checks

PostgreSQL health checks ensure the application waits until the database is ready.

```yaml
healthcheck:
  test: ["CMD", "pg_isready", "-U", "admin", "-d", "mydb"]
  interval: 10s
  timeout: 5s
  retries: 5
```

---

## Docker Hub Deployment

### Tag Image

```bash
docker tag node-app-multi sopatel264/node-app-multi:latest
```

### Push Image

```bash
docker push sopatel264/node-app-multi:latest
```

Docker Hub Repository:

```text
https://hub.docker.com/r/sopatel264/node-app-multi
```

---

## Testing Fresh Deployment

To ensure the application works from a clean environment:

```bash
docker compose down
docker image rm sopatel264/node-app-multi:latest
docker compose up -d
```

Docker automatically pulled the image from Docker Hub and started all services successfully.

---

## Challenges Faced

### PostgreSQL Startup Timing

**Issue**

The application attempted to start before PostgreSQL was ready.

**Solution**

Implemented a PostgreSQL health check and configured:

```yaml
depends_on:
  db:
    condition: service_healthy
```

---

### Environment Variable Management

**Issue**

Hardcoded credentials are not recommended.

**Solution**

Moved database configuration into `.env` and committed only `.env.example`.

---

### Docker Hub Deployment

**Issue**

Needed to verify the application could run without local images.

**Solution**

Removed local images and tested deployment directly from Docker Hub.

---

## Screenshots

### Project Structure

<img width="1096" height="806" alt="image" src="https://github.com/user-attachments/assets/e147b450-d7aa-450d-be2f-6d95b31adb8b" />


### Docker Compose Services

<img width="953" height="244" alt="image" src="https://github.com/user-attachments/assets/60abdfe6-1eb1-4f3c-b29b-5687f4897375" />

### Running Containers

<img width="1156" height="149" alt="image" src="https://github.com/user-attachments/assets/e9f4b032-53d1-4f4f-ae23-b496f9f71456" />

### Application Running

<img width="1589" height="906" alt="image" src="https://github.com/user-attachments/assets/2c97fd02-387c-4ba2-bfd0-0f02e63fb346" />

### Docker Hub Repository

<img width="1678" height="411" alt="image" src="https://github.com/user-attachments/assets/8403e671-4be4-44dd-a12c-1d625e6e5951" />

### Docker Volumes

<img width="616" height="173" alt="image" src="https://github.com/user-attachments/assets/c6cf6e7c-17a8-4036-aa75-5847c75886ea" />

### Docker Networks

<img width="610" height="222" alt="image" src="https://github.com/user-attachments/assets/7f976174-e3e1-4dd3-8e50-7a56836c7613" />


### Container Logs

<img width="934" height="541" alt="image" src="https://github.com/user-attachments/assets/1c2ef464-2def-4933-93be-2856f3fa3c30" />

---

## Verification Results

* ✅ Node.js application container running
* ✅ PostgreSQL container healthy
* ✅ Redis container running
* ✅ Docker Compose orchestration successful
* ✅ Persistent volume configured
* ✅ Custom Docker network configured
* ✅ Environment variables externalized
* ✅ Docker Hub image pushed successfully
* ✅ Fresh deployment tested successfully

---

## Key Learnings

* Dockerizing real-world applications
* Writing production-ready Dockerfiles
* Multi-container deployments with Docker Compose
* PostgreSQL integration
* Redis integration
* Health checks and service dependencies
* Docker networking
* Persistent storage with volumes
* Environment variable management
* Publishing images to Docker Hub

---

## Conclusion

Successfully Dockerized a complete Node.js application using Docker, PostgreSQL, Redis, Docker Compose, health checks, custom networking, persistent storage, and Docker Hub. The application can now be deployed consistently across environments using a single Docker Compose command.
