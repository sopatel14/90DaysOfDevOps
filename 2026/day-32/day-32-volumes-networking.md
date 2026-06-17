# Day 32 - Docker Volumes & Networking

## Objective

Learn how Docker handles persistent storage and container communication using:

* Named Volumes
* Bind Mounts
* Docker Networks
* Custom Bridge Networks

---

# Task 1: The Problem (Data Loss)

## Run PostgreSQL Container

```bash
docker run -d \
--name postgres-test \
-e POSTGRES_PASSWORD=password \
postgres
```

## Access Container

```bash
docker exec -it postgres-test psql -U postgres
```

## Create Database Table

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50)
);

INSERT INTO users(name)
VALUES ('Sourav');

SELECT * FROM users;
```

<img width="567" height="312" alt="image" src="https://github.com/user-attachments/assets/6a398491-da00-49b5-aec5-667133eab5e8" />


## Stop and Remove Container

```bash
docker stop postgres-test
docker rm postgres-test
```

## Create New Container

```bash
docker run -d \
--name postgres-test-new \
-e POSTGRES_PASSWORD=password \
postgres
```

## Verify Data

```bash
docker exec -it postgres-test-new psql -U postgres
```

```sql
SELECT * FROM users;
```

### Result

The table does not exist.

<img width="571" height="245" alt="image" src="https://github.com/user-attachments/assets/e0318225-e723-40db-9043-dae80d8bfa97" />


### Why?

Containers are ephemeral.

When the original container was removed, all writable container data was removed as well.

Without persistent storage, container data is lost.

---

# Task 2: Named Volumes

## Create Volume

```bash
docker volume create postgres-data
```

## Verify

```bash
docker volume ls
```
<img width="557" height="183" alt="image" src="https://github.com/user-attachments/assets/1a5b8812-29e5-46a6-86cf-2a915315e7fa" />


## Run PostgreSQL with Volume

```bash
docker run -d \
--name postgres-volume \
-e POSTGRES_PASSWORD=password \
-v postgres-data:/var/lib/postgresql/data \
postgres
```

## Create Data

```bash
docker exec -it postgres-volume psql -U postgres
```

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50)
);

INSERT INTO employees(name)
VALUES ('Docker User');

SELECT * FROM employees;
```

<img width="570" height="376" alt="image" src="https://github.com/user-attachments/assets/7552899f-1db4-46b5-a6fa-b6a747bc115f" />


## Remove Container

```bash
docker stop postgres-volume
docker rm postgres-volume
```

## Start New Container Using Same Volume

```bash
docker run -d \
--name postgres-volume-new \
-e POSTGRES_PASSWORD=password \
-v postgres-data:/var/lib/postgresql/data \
postgres
```

## Verify Data

```bash
docker exec -it postgres-volume-new psql -U postgres
```

```sql
SELECT * FROM employees;
```

<img width="571" height="366" alt="image" src="https://github.com/user-attachments/assets/a91d981e-81e7-4da0-91de-3d0a69e476f1" />


### Result

The data still exists.

### Why?

Named volumes are stored outside the container lifecycle.

Removing a container does not remove the volume.

## Inspect Volume

```bash
docker volume inspect postgres-data
```
<img width="565" height="248" alt="image" src="https://github.com/user-attachments/assets/f0ad4561-c888-4644-99c0-73859a973af8" />


---

# Task 3: Bind Mounts

## Create Host Directory

```bash
mkdir nginx-site
cd nginx-site
```

## Create index.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>Docker Bind Mount</title>
</head>
<body>
    <h1>Hello from Bind Mount!</h1>
</body>
</html>
```

## Run Nginx Container

```bash
docker run -d \
--name nginx-bind \
-p 8080:80 \
-v $(pwd):/usr/share/nginx/html \
nginx
```

## Verify

Open browser:

```text
http://localhost:8080
```

<img width="558" height="344" alt="image" src="https://github.com/user-attachments/assets/8f4c10f2-3ec1-4135-bee9-16ecbe59bf3f" />


Page displays:

```text
Hello from Bind Mount!
```

## Modify index.html

```html
<h1>Updated Content!</h1>
```

Refresh browser.

<img width="539" height="351" alt="image" src="https://github.com/user-attachments/assets/bf758a78-c0a7-4f9d-928d-56f123955722" />


### Result

Changes appear immediately.

### Named Volume vs Bind Mount

| Named Volume                  | Bind Mount                             |
| ----------------------------- | -------------------------------------- |
| Managed by Docker             | Managed by Host OS                     |
| Stored in Docker storage area | Stored in any host directory           |
| Better for databases          | Better for source code and development |
| Portable                      | Depends on host filesystem             |

---

# Task 4: Docker Networking Basics

## List Networks

```bash
docker network ls
```

Example:

```text
bridge
host
none
```

<img width="540" height="123" alt="image" src="https://github.com/user-attachments/assets/ce6c5f5d-b33d-473d-8f65-7554857c0c91" />


## Inspect Bridge Network

```bash
docker network inspect bridge
```

## Run Two Containers

```bash
docker run -dit --name container1 ubuntu
docker run -dit --name container2 ubuntu
```

## Install Ping Utility

```bash
docker exec container1 apt update
docker exec container1 apt install -y iputils-ping
```

## Ping by Name

```bash
docker exec -it container1 ping container2
```

<img width="566" height="88" alt="image" src="https://github.com/user-attachments/assets/942c015d-face-40aa-9484-ebb5342ec065" />


### Result

Fails.

## Ping by IP

Find container2 IP:

```bash
docker inspect container2
```

Ping using IP:

```bash
docker exec -it container1 ping <container2-ip>
```

<img width="567" height="237" alt="image" src="https://github.com/user-attachments/assets/e491c8b9-6c03-4ea6-85c3-a7b3d6f3e168" />


### Result

Works.

### Why?

Containers on the default bridge network do not automatically receive DNS-based name resolution.

---

# Task 5: Custom Networks

## Create Network

```bash
docker network create my-app-net
```

## Run Containers

```bash
docker run -dit \
--name app1 \
--network my-app-net \
ubuntu

docker run -dit \
--name app2 \
--network my-app-net \
ubuntu
```

## Install Ping

```bash
docker exec app1 apt update
docker exec app1 apt install -y iputils-ping
```

## Ping by Name

```bash
docker exec -it app1 ping app2
```

<img width="559" height="206" alt="image" src="https://github.com/user-attachments/assets/f65b43cc-5898-4830-a24f-6c52b1d5023e" />


### Result

Works successfully.

### Why?

User-defined bridge networks include an internal DNS server.

Docker automatically resolves container names to container IP addresses.

This allows containers to communicate using names instead of IP addresses.

---

# Task 6: Put It Together

## Create Network

```bash
docker network create app-db-net
```

## Create Volume

```bash
docker volume create db-data
```

## Run PostgreSQL

```bash
docker run -d \
--name postgres-db \
--network app-db-net \
-v db-data:/var/lib/postgresql/data \
-e POSTGRES_PASSWORD=password \
postgres
```

## Run Application Container

```bash
docker run -dit \
--name app-container \
--network app-db-net \
ubuntu
```

## Install Networking Tools

```bash
docker exec app-container apt update
docker exec app-container apt install -y iputils-ping
```

## Test Connectivity

```bash
docker exec -it app-container ping postgres-db
```

<img width="643" height="266" alt="image" src="https://github.com/user-attachments/assets/4f2257d2-b688-4d7c-a9d5-7b4fd712be08" />


### Result

Ping succeeds.

### Verification

* Database persists using volume.
* Containers communicate using container names.
* Both containers share the same custom network.

---

# Key Commands Learned

```bash
docker volume create
docker volume ls
docker volume inspect

docker network create
docker network ls
docker network inspect

docker run -v
docker run --network

docker exec

docker inspect
```

---

# Key Takeaways

* Containers lose data when removed.
* Named volumes provide persistent storage.
* Bind mounts connect host files directly to containers.
* Default bridge networking allows communication by IP.
* Custom bridge networks provide automatic DNS resolution.
* Volumes and custom networks are essential for real-world applications.
* Multi-container applications rely on both persistent storage and networking.
