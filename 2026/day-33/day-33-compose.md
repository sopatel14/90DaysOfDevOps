# Day 33 – Docker Compose: Multi-Container Basics

## Objective

Learn how to use Docker Compose to manage multi-container applications using a single YAML configuration file.

---

# Task 1: Install & Verify Docker Compose

## Check Docker Compose Availability

```bash
docker compose version
```

### Output

```bash
Docker Compose version v2.x.x
```

## Verify Docker Version

```bash
docker --version
```

### Output

```bash
Docker version 28.x.x
```

---

# Task 2: First Docker Compose File

## Create Project Directory

```bash
mkdir compose-basics
cd compose-basics
```

## Create docker-compose.yml

```yaml
services:
  nginx:
    image: nginx:latest
    container_name: nginx-compose
    ports:
      - "8080:80"
```

## Start Container

```bash
docker compose up 
```


# Day 33 – Docker Compose: Multi-Container Basics

## Objective

Learn how to use Docker Compose to manage multi-container applications using a single YAML configuration file.

---

# Task 1: Install & Verify Docker Compose

## Check Docker Compose Availability

```bash
docker compose version
```

### Output

```bash
Docker Compose version v2.x.x
```

## Verify Docker Version

```bash
docker --version
```

### Output

```bash
Docker version 28.x.x
```

---

# Task 2: First Docker Compose File

## Create Project Directory

```bash
mkdir compose-basics
cd compose-basics
```

## Create docker-compose.yml

```yaml
services:
  nginx:
    image: nginx:latest
    container_name: nginx-compose
    ports:
      - "8080:80"
```

## Start Container

```bash
docker compose up 
```
<img width="1126" height="832" alt="image" src="https://github.com/user-attachments/assets/8120e422-2320-40b9-a808-60f61503c346" />


## Verify Running Containers

```bash
docker ps
```

<img width="569" height="91" alt="image" src="https://github.com/user-attachments/assets/0f0481da-03d4-4bcc-a9df-1e2e69473063" />


## Access Nginx

Open browser:

```text
http://<EC2-Public-IP>:8080
```

<img width="1229" height="465" alt="image" src="https://github.com/user-attachments/assets/edf2a475-aaf6-41c4-8a00-bddf8443c73f" />


You should see the Nginx welcome page.

## Stop and Remove

```bash
docker compose down
```

<img width="567" height="92" alt="image" src="https://github.com/user-attachments/assets/8be61827-f6c5-4c62-a760-a415a6e9c746" />


---

# Task 3: WordPress + MySQL Setup

## Create Project Directory

```bash
mkdir wordpress-compose
cd wordpress-compose
```

## Create docker-compose.yml

```yaml
services:

  db:
    image: mysql:8.0
    container_name: mysql-db
    restart: always
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppassword
      MYSQL_ROOT_PASSWORD: rootpassword
    volumes:
      - mysql_data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    container_name: wordpress-app
    restart: always
    ports:
      - "8081:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppassword
      WORDPRESS_DB_NAME: wordpress
    depends_on:
      - db

volumes:
  mysql_data:
```

## Start Services

```bash
docker compose up -d
```

<img width="565" height="114" alt="image" src="https://github.com/user-attachments/assets/35fc51d2-e6be-408d-8ea2-9dba47d97a71" />


## Verify Running Containers

```bash
docker ps
```
<img width="569" height="141" alt="image" src="https://github.com/user-attachments/assets/19c7ef49-91ba-485e-9ca6-c637b398c463" />


## Access WordPress

Open browser:

```text
http://<EC2-Public-IP>:8081
```
<img width="970" height="834" alt="image" src="https://github.com/user-attachments/assets/ec1958c3-d02a-4329-b66f-36b73b206fab" />

<img width="780" height="286" alt="image" src="https://github.com/user-attachments/assets/839b515e-a725-4a40-936e-e1c39faadfc2" />


Complete the WordPress setup wizard.

## Verify Persistence

Stop everything:

```bash
docker compose down
```

Start again:

```bash
docker compose up -d
```

Visit WordPress again.

<img width="1680" height="895" alt="image" src="https://github.com/user-attachments/assets/243f0b48-10bd-4d33-9aed-7a4c46ce452d" />


Result:

* Website still exists
* Database data persisted
* Named volume retained data successfully

---

# Task 4: Docker Compose Commands

## Start Services

```bash
docker compose up -d
```

## View Running Services

```bash
docker compose ps
```

## View Logs of All Services

```bash
docker compose logs
```

## Follow Logs Continuously

```bash
docker compose logs -f
```

## View Logs for Specific Service

```bash
docker compose logs wordpress
```

or

```bash
docker compose logs db
```

## Stop Services Without Removing

```bash
docker compose stop
```

## Start Stopped Services

```bash
docker compose start
```

## Remove Containers and Networks

```bash
docker compose down
```

## Rebuild Images

```bash
docker compose up --build
```

---

# Task 5: Environment Variables

## Create .env File

```env
MYSQL_DATABASE=wordpress
MYSQL_USER=wpuser
MYSQL_PASSWORD=wppassword
MYSQL_ROOT_PASSWORD=rootpassword
```

## Update docker-compose.yml

```yaml
services:

  db:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
```

## Start Services

```bash
docker compose up -d
```

## Verify Environment Variables

```bash
docker compose exec db env
```

<img width="538" height="244" alt="image" src="https://github.com/user-attachments/assets/70503a1f-e4b5-4cbe-b15b-d9bb36198cd5" />


or

```bash
docker inspect mysql-db
```

Result:

Docker Compose successfully reads values from the `.env` file.

---

# Key Learnings

* Docker Compose allows multiple containers to be managed from a single YAML file.
* Services automatically communicate through a default network.
* Service names act as DNS names inside the Compose network.
* Named volumes provide persistent storage.
* Environment variables can be hardcoded or loaded from a `.env` file.
* Docker Compose simplifies deployment and management of multi-container applications.

---




