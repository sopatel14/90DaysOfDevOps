# Day 30 – Docker Images & Container Lifecycle

## Objective

Learn how Docker images and containers work, understand image layers and caching, and practice the complete container lifecycle.

---

# Task 1: Docker Images

## Pull Images from Docker Hub

Pull Nginx:

```bash
docker pull nginx
```

<img width="583" height="312" alt="image" src="https://github.com/user-attachments/assets/4d1d9057-e5a2-4167-ab27-a85ad4968d10" />


Pull Ubuntu:

```bash
docker pull ubuntu
```

<img width="612" height="198" alt="image" src="https://github.com/user-attachments/assets/9a1c9c50-1eb6-4314-b8ce-fae731f55b4d" />


Pull Alpine:

```bash
docker pull alpine
```

<img width="613" height="161" alt="image" src="https://github.com/user-attachments/assets/9c570a1d-bbcf-4123-8f13-4525f2ec4828" />


---

## List All Images

```bash
docker images
```

Example Output:

<img width="795" height="203" alt="image" src="https://github.com/user-attachments/assets/d19a053b-d23d-45a9-8381-616c07fbbcb4" />


---

## Ubuntu vs Alpine

### Ubuntu

* Full Linux distribution
* Includes many utilities and packages
* Easier for beginners
* Larger image size

### Alpine

* Minimal Linux distribution
* Designed for containers
* Smaller attack surface
* Faster downloads
* Much smaller image size

### Why is Alpine Smaller?

Alpine uses BusyBox and only includes essential components required to run applications, making it significantly smaller than Ubuntu.

---

## Inspect an Image

Inspect Nginx:

```bash
docker image inspect nginx
```

Information available:

* Image ID
* Architecture
* Environment variables
* Creation date
* Entrypoint
* Working directory
* Labels
* Layers

---

## Remove an Image

```bash
docker rmi alpine
```

Verify:

```bash
docker images
```

<img width="792" height="236" alt="image" src="https://github.com/user-attachments/assets/543f6491-f01a-4118-8687-a435c47d22e0" />


---

# Task 2: Image Layers

## View Image History

```bash
docker image history nginx
```

Example:

<img width="788" height="369" alt="image" src="https://github.com/user-attachments/assets/2a3a29ae-7922-457e-9827-7452bbf5c58f" />



---

## What Are Docker Layers?

Each instruction in a Dockerfile creates a new layer.

Examples:

```dockerfile
FROM ubuntu
RUN apt update
RUN apt install nginx
COPY . .
```

Each line creates a separate layer.

---

## Why Does Docker Use Layers?

### Benefits

* Faster builds
* Better caching
* Reduced storage usage
* Easier image sharing

If a layer already exists, Docker reuses it instead of rebuilding it.

---

# Task 3: Container Lifecycle

## Create a Container Without Starting

```bash
docker create --name lifecycle-demo nginx
```

Check:

```bash
docker ps -a
```

<img width="793" height="260" alt="image" src="https://github.com/user-attachments/assets/707369fe-4128-41c1-beaa-f12be06eaf56" />


State:

```text
Created
```

---

## Start the Container

```bash
docker start lifecycle-demo
```

State:

```text
Up
```
<img width="776" height="139" alt="image" src="https://github.com/user-attachments/assets/a4013f58-bcd6-48ba-81c5-38839cbcb037" />


---

## Pause the Container

```bash
docker pause lifecycle-demo
```

State:

```text
Paused
```

<img width="799" height="141" alt="image" src="https://github.com/user-attachments/assets/da2e9bbe-a6bd-4b19-9736-0b7645b3a161" />


---

## Unpause the Container

```bash
docker unpause lifecycle-demo
```

State:

```text
Up
```

<img width="793" height="116" alt="image" src="https://github.com/user-attachments/assets/c7209a35-f3e5-43b9-b749-888d0398d6a3" />


---

## Stop the Container

```bash
docker stop lifecycle-demo
```

State:

```text
Exited
```

<img width="677" height="94" alt="image" src="https://github.com/user-attachments/assets/31284a87-c36a-40f3-a221-a7b4c836cd84" />


---

## Restart the Container

```bash
docker restart lifecycle-demo
```

State:

```text
Up
```

<img width="792" height="114" alt="image" src="https://github.com/user-attachments/assets/0d0833d1-783a-44fe-9226-c1911b9be8a5" />


---

## Kill the Container

```bash
docker kill lifecycle-demo
```

State:

```text
Exited
```

<img width="617" height="95" alt="image" src="https://github.com/user-attachments/assets/c0ae8ce9-ed6d-4b5e-bbf3-8f2ccb1517ab" />


---

## Remove the Container

```bash
docker rm lifecycle-demo
```

Verify:

```bash
docker ps -a
```

Container should no longer exist.

<img width="790" height="223" alt="image" src="https://github.com/user-attachments/assets/771f893a-5e49-4a5d-8a20-ff91298c09e5" />



---

## Container Lifecycle Diagram

```text
Create
  |
  v
Created
  |
Start
  |
  v
Running
  |
Pause
  |
  v
Paused
  |
Unpause
  |
  v
Running
  |
Stop/Kill
  |
  v
Exited
  |
Remove
  |
  v
Deleted
```

---

# Task 4: Working with Running Containers

## Run Nginx in Detached Mode

```bash
docker run -d --name nginx-demo -p 8080:80 nginx
```

Verify:

```bash
docker ps
```

<img width="801" height="172" alt="image" src="https://github.com/user-attachments/assets/d364df4a-edbe-4c94-b68f-5655a11f7a6f" />


---

## View Logs

```bash
docker logs nginx-demo
```

Displays container output.

<img width="719" height="341" alt="image" src="https://github.com/user-attachments/assets/85a8ea77-4b5f-4324-aca9-b82661a0b389" />


---

## Follow Logs in Real Time

```bash
docker logs -f nginx-demo
```

<img width="710" height="326" alt="image" src="https://github.com/user-attachments/assets/90307934-05cc-47c9-b6d8-99cb4e3ff654" />


Press:

```text
CTRL + C
```

to exit.

---

## Enter the Container

```bash
docker exec -it nginx-demo bash
```

Explore:

```bash
pwd
ls
cd /usr/share/nginx/html
ls
```


<img width="791" height="198" alt="image" src="https://github.com/user-attachments/assets/38ac6b34-e4c5-47e4-86ab-9730712b39a0" />


Exit:

```bash
exit
```

---

## Run a Single Command

```bash
docker exec nginx-demo hostname
```

Example Output:

```text
6a12b34cd56
```

<img width="671" height="76" alt="image" src="https://github.com/user-attachments/assets/028cfaba-2150-4841-841f-87ba1ddd41f0" />


---

## Inspect Container

```bash
docker inspect nginx-demo
```

Useful information:

* Container IP address
* Port mappings
* Mounts
* Network settings
* Environment variables

Find only the IP:

```bash
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' nginx-demo
```

<img width="796" height="90" alt="image" src="https://github.com/user-attachments/assets/0eb67057-de3e-4a66-98fa-5e1ffdf33034" />


---

# Task 5: Cleanup

## Stop All Running Containers

```bash
docker stop $(docker ps -q)
```

---

## Remove All Stopped Containers

```bash
docker container prune -f
```

<img width="620" height="93" alt="image" src="https://github.com/user-attachments/assets/1007ff63-8da3-4f0a-a2fa-a75bd0ce0335" />


---

## Remove Unused Images

```bash
docker image prune -a -f
```

---

## Check Docker Disk Usage

```bash
docker system df
```

<img width="486" height="149" alt="image" src="https://github.com/user-attachments/assets/18a3612a-73c2-4197-ab90-27340d12dd19" />


Example:

```text
TYPE            TOTAL     ACTIVE    SIZE
Images          5         2         600MB
Containers      4         1         50MB
Volumes         2         1         20MB
```

---

## Full Docker Cleanup

```bash
docker system prune -a
```

Removes:

* Stopped containers
* Unused images
* Unused networks
* Build cache

---

# Key Takeaways

* Images are templates used to create containers.
* Containers are running instances of images.
* Docker images are built from layers.
* Layers improve caching and reduce storage usage.
* Containers move through a lifecycle: Created → Running → Paused → Exited → Removed.
* Docker logs help troubleshoot running containers.
* Docker inspect provides detailed container information.
* Cleanup commands are important to manage disk space efficiently.

---

## Commands Practiced Today

```bash
docker pull
docker images
docker image inspect
docker image history
docker rmi

docker create
docker start
docker pause
docker unpause
docker stop
docker restart
docker kill
docker rm

docker run -d
docker logs
docker logs -f
docker exec
docker inspect

docker container prune
docker image prune
docker system df
docker system prune
```
