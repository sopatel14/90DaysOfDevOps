# Day 29 – Introduction to Docker

## Objective

Learn the fundamentals of Docker, understand containers, install Docker, and run your first containers.

---

# Task 1: What is Docker?

## What is a Container?

A container is a lightweight, portable package that contains everything an application needs to run:

* Application code
* Runtime
* Libraries
* Dependencies
* Configuration files

Containers ensure that applications run consistently across different environments.

### Why Do We Need Containers?

Before containers, applications often worked on one machine but failed on another because of dependency or configuration differences.

Containers solve this problem by packaging everything together, making deployments predictable and reliable.

---

## Containers vs Virtual Machines

| Feature        | Containers           | Virtual Machines       |
| -------------- | -------------------- | ---------------------- |
| OS Required    | Share host OS kernel | Each VM has its own OS |
| Startup Time   | Seconds              | Minutes                |
| Resource Usage | Lightweight          | Heavy                  |
| Performance    | Near-native          | Higher overhead        |
| Size           | MBs                  | GBs                    |
| Isolation      | Process-level        | Hardware-level         |

### Simple Explanation

A Virtual Machine is like having multiple houses on the same land.

A Container is like having multiple apartments inside one building sharing the same infrastructure.

---

## Docker Architecture

Docker consists of four major components:

### Docker Client

The command-line interface used by users.

Examples:

```bash
docker run
docker ps
docker stop
```

---

### Docker Daemon (dockerd)

The background service responsible for:

* Building images
* Running containers
* Managing networks
* Managing storage

---

### Docker Images

Read-only templates used to create containers.

Examples:

* nginx
* ubuntu
* mysql

---

### Docker Containers

Running instances of Docker images.

Example:

```bash
docker run nginx
```

This creates and starts a container from the Nginx image.

---

### Docker Registry

A repository where Docker images are stored.

Most commonly:

* Docker Hub

Website:

https://hub.docker.com

---

## Docker Architecture Diagram

```text
+------------------+
|  Docker Client   |
| (docker command) |
+--------+---------+
         |
         v
+------------------+
| Docker Daemon    |
|    (dockerd)     |
+----+--------+----+
     |        |
     v        v
 Images   Containers
     |
     v
 Docker Hub Registry
```

### In My Own Words

When a user runs a Docker command, the Docker Client sends the request to the Docker Daemon. The daemon pulls images from Docker Hub if necessary and creates containers from those images.

---

# Task 2: Install Docker

## Install Docker

Update packages:

```bash
sudo apt update
```

Install Docker:

```bash
sudo apt install docker.io -y
```
### Output

<img width="666" height="429" alt="image" src="https://github.com/user-attachments/assets/35972fdc-61a2-4dc9-b929-21b91751fa44" />


Enable Docker:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Check status:

```bash
sudo systemctl status docker
```

### Output

<img width="667" height="461" alt="image" src="https://github.com/user-attachments/assets/aa8c9849-93e9-48f5-9ecd-14fbb1a80413" />


---

## Verify Installation

Check Docker version:

```bash
docker --version
```

### Output

<img width="472" height="64" alt="image" src="https://github.com/user-attachments/assets/49108eec-47df-4484-ac97-802b42c150d9" />


---

## Run Hello World

```bash
docker run hello-world
```

### Output

<img width="649" height="546" alt="image" src="https://github.com/user-attachments/assets/0926dc62-2fdd-4ed3-8477-343fa179f5ed" />


Output confirms:

* Docker is installed correctly
* Image was downloaded
* Container executed successfully

---

# Task 3: Run Real Containers

## Run an Nginx Container

```bash
docker run -d -p 80:80 --name nginx-container nginx
```


Explanation:

* -d = detached mode
* -p 8080:80 = port mapping
* --name = custom container name

---

## Verify Running Containers

```bash
sudo docker ps
```

### Output

<img width="667" height="167" alt="image" src="https://github.com/user-attachments/assets/1f2922bb-dd2d-4d42-bfe9-b162f5681bc9" />


---

## Access Nginx

Browser:

<img width="1168" height="433" alt="image" src="https://github.com/user-attachments/assets/b65a82bf-f6e4-4a23-924a-a697e16df28e" />


---

## Run Ubuntu in Interactive Mode

```bash
docker run -it ubuntu
```

Inside the container:

```bash
ls
pwd
cat /etc/os-release
```

### Output

<img width="658" height="471" alt="image" src="https://github.com/user-attachments/assets/bc8e754c-43a4-4334-b5bf-6cb54f291501" />


---

## List Running Containers

```bash
sudo docker ps
```
### Ouptut

<img width="670" height="127" alt="image" src="https://github.com/user-attachments/assets/1a92d67e-0061-42df-bd28-965b378f6da4" />

---

## List All Containers

```bash
sudo docker ps -a
```
### Output

<img width="662" height="177" alt="image" src="https://github.com/user-attachments/assets/dc9f1990-8c82-4f70-ad8a-68f11923a8aa" />


---

## Stop a Container

```bash
docker stop container id
```

---

## Remove a Container

```bash
docker rm container id
```

---

# Task 4: Explore Docker

## Detached Mode

Run:

```bash
docker run -d nginx
```

### What is Different?

Detached mode runs the container in the background and immediately returns control to the terminal.

Without detached mode, logs are displayed directly in the terminal.

---

## Give a Container a Custom Name

```bash
docker run -d --name my-nginx nginx
```

Verify:

```bash
sudo docker ps
```

---

## Port Mapping

```bash
sudo docker run -d -p 8081:80 nginx
```

Meaning:

* Host Port: 8081
* Container Port: 80

Access:

```text
http://<server-ip>:8081
```
<img width="1059" height="412" alt="image" src="https://github.com/user-attachments/assets/01ca4681-c9f4-4c04-8c76-fe1cd03643fa" />



---

## View Container Logs

```bash
docker logs my-nginx
```
### Output

<img width="672" height="367" alt="image" src="https://github.com/user-attachments/assets/1b833654-0cfb-4e3b-9df0-eded2cb258a1" />



Useful for:

* Troubleshooting
* Debugging applications
* Monitoring startup events

---

## Execute Commands Inside a Running Container

```bash
docker exec -it my-nginx bash
```

Examples:

```bash
ls
pwd
hostname
```

### Output

<img width="658" height="184" alt="image" src="https://github.com/user-attachments/assets/ee2ee9c9-55a8-47e1-b402-f2a9e46a51c6" />


Exit:

```bash
exit
```

---

# Common Docker Commands

```bash
docker --version
docker run hello-world
docker run -d nginx
docker run -it ubuntu
docker ps
docker ps -a
docker stop <container>
docker rm <container>
docker logs <container>
docker exec -it <container> bash
```

---

# Key Takeaways

* Docker packages applications into containers.
* Containers are lightweight compared to virtual machines.
* Docker Images are templates used to create containers.
* Containers are running instances of images.
* Docker Hub stores and distributes images.
* Nginx can be deployed in seconds using Docker.
* Detached mode allows containers to run in the background.
* Port mapping exposes container services to users.
* Docker is a foundational technology for modern DevOps, CI/CD, and Kubernetes environments.

---
