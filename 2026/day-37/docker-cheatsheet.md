# Docker Cheat Sheet

## Container Commands

| Command                            | Purpose                        |
| ---------------------------------- | ------------------------------ |
| `docker run nginx`                 | Run a container                |
| `docker run -d nginx`              | Run container in detached mode |
| `docker run -it ubuntu bash`       | Interactive container          |
| `docker ps`                        | List running containers        |
| `docker ps -a`                     | List all containers            |
| `docker stop <container>`          | Stop a container               |
| `docker start <container>`         | Start a stopped container      |
| `docker restart <container>`       | Restart a container            |
| `docker rm <container>`            | Remove a container             |
| `docker exec -it <container> bash` | Access container shell         |
| `docker logs <container>`          | View container logs            |

---

## Image Commands

| Command                          | Purpose                    |
| -------------------------------- | -------------------------- |
| `docker pull nginx`              | Pull image from Docker Hub |
| `docker images`                  | List local images          |
| `docker build -t myapp .`        | Build image                |
| `docker tag myapp user/myapp:v1` | Tag image                  |
| `docker push user/myapp:v1`      | Push image to Docker Hub   |
| `docker image rm <image>`        | Remove image               |

---

## Volume Commands

| Command                          | Purpose        |
| -------------------------------- | -------------- |
| `docker volume create myvolume`  | Create volume  |
| `docker volume ls`               | List volumes   |
| `docker volume inspect myvolume` | Inspect volume |
| `docker volume rm myvolume`      | Remove volume  |

---

## Network Commands

| Command                                    | Purpose                      |
| ------------------------------------------ | ---------------------------- |
| `docker network create backend`            | Create network               |
| `docker network ls`                        | List networks                |
| `docker network inspect backend`           | Inspect network              |
| `docker network connect backend container` | Connect container to network |

---

## Docker Compose Commands

| Command                  | Purpose          |
| ------------------------ | ---------------- |
| `docker compose up -d`   | Start services   |
| `docker compose down`    | Stop services    |
| `docker compose ps`      | View services    |
| `docker compose logs`    | View logs        |
| `docker compose build`   | Build services   |
| `docker compose restart` | Restart services |

---

## Cleanup Commands

| Command                  | Purpose                     |
| ------------------------ | --------------------------- |
| `docker container prune` | Remove stopped containers   |
| `docker image prune`     | Remove unused images        |
| `docker volume prune`    | Remove unused volumes       |
| `docker network prune`   | Remove unused networks      |
| `docker system prune -a` | Remove all unused resources |
| `docker system df`       | Show Docker disk usage      |

---

## Dockerfile Instructions

| Instruction  | Purpose                         |
| ------------ | ------------------------------- |
| `FROM`       | Base image                      |
| `RUN`        | Execute commands during build   |
| `COPY`       | Copy files into image           |
| `ADD`        | Copy files and extract archives |
| `WORKDIR`    | Set working directory           |
| `EXPOSE`     | Document listening port         |
| `ENV`        | Set environment variable        |
| `CMD`        | Default command                 |
| `ENTRYPOINT` | Fixed executable                |
| `USER`       | Run container as specific user  |

---

## Common Port Mapping

```bash
docker run -p 8080:80 nginx
```

Maps:

* Host Port: 8080
* Container Port: 80

Access:

```text
http://localhost:8080
```

---

## Useful Inspection Commands

```bash
docker inspect <container>
docker stats
docker system df
docker top <container>
docker history <image>
```
