# Day 31 - Dockerfile: Build Your Own Images

## Objective

Learn how to create custom Docker images using Dockerfiles, understand common Dockerfile instructions, compare CMD and ENTRYPOINT, build a simple web application image, use .dockerignore, and optimize image builds using layer caching.

---

# Task 1: Your First Dockerfile

## Dockerfile

```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get install -y curl

CMD ["echo", "Hello from my custom image!"]
```

## Build Image

```bash
docker build -t my-ubuntu:v1 .
```

## Run Container

```bash
docker run my-ubuntu:v1
```

## Output

<img width="1390" height="174" alt="image" src="https://github.com/user-attachments/assets/628186e1-0ac0-402e-9457-bdeefc36f4fa" />


### What Happened?

- FROM downloads Ubuntu as the base image.
- RUN installs curl during image build.
- CMD defines the default command executed when the container starts.

---

# Task 2: Dockerfile Instructions

## app.txt

```text
Dockerfile Instructions Demo
```

## Dockerfile

```dockerfile
FROM ubuntu:latest

RUN apt-get update

WORKDIR /app

COPY app.txt .

EXPOSE 8080

CMD ["cat", "app.txt"]
```

## Build

```bash
docker build -t instructions-demo:v1 .
```

## Run

```bash
docker run instructions-demo:v1
```

<img width="678" height="71" alt="image" src="https://github.com/user-attachments/assets/293f03f3-2cda-410c-a5f9-ae7e496439db" />


## Explanation

### FROM

Defines the base image.

### RUN

Executes commands during image build.

### WORKDIR

Sets the working directory inside the container.

### COPY

Copies files from host machine to image.

### EXPOSE

Documents the application's listening port.

### CMD

Specifies the default command executed when the container starts.

---

# Task 3: CMD vs ENTRYPOINT

## CMD Example

### Dockerfile

```dockerfile
FROM ubuntu:latest

CMD ["echo", "hello"]
```

### Run Normally

```bash
docker run cmd-demo
```

Output:

<img width="712" height="325" alt="image" src="https://github.com/user-attachments/assets/f66edeb7-62db-4502-8c57-a63061e7bc96" />


### Override CMD

```bash
docker run cmd-demo ls
```

Output:

<img width="636" height="358" alt="image" src="https://github.com/user-attachments/assets/535f256d-eb1a-47fc-ba25-6aa1b0fd12fa" />


### Observation

CMD can be completely replaced by a command supplied during docker run.

---

## ENTRYPOINT Example

### Dockerfile

```dockerfile
FROM ubuntu:latest

ENTRYPOINT ["echo"]
```

### Run Normally

```bash
docker run entrypoint-demo hello
```

Output:

```text
hello
```

### Run With Additional Arguments

```bash
docker run entrypoint-demo Docker Rocks!
```

Output:

<img width="709" height="110" alt="image" src="https://github.com/user-attachments/assets/769cf4b9-15e5-48c3-8906-fdd3a05b9d94" />


### Observation

ENTRYPOINT remains fixed and additional arguments are appended to it.

---

## CMD vs ENTRYPOINT

| CMD | ENTRYPOINT |
|------|------------|
| Provides default command | Provides fixed executable |
| Easily overridden | Difficult to override |
| Best for flexible containers | Best for dedicated containers |

### When to Use CMD

Use CMD when users may want to run different commands.

### When to Use ENTRYPOINT

Use ENTRYPOINT when the container should always run a specific application.

---

# Task 4: Build a Simple Web App Image

## index.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>My Docker Website</title>
</head>
<body>
    <h1>Hello from Docker!</h1>
    <p>Day 31 Dockerfile Challenge</p>
</body>
</html>
```

## Dockerfile

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html
```

## Build

```bash
docker build -t my-website:v1 .
```

## Run

```bash
docker run -d -p 8080:80 my-website:v1
```

## Verify

Open browser:

<img width="494" height="202" alt="image" src="https://github.com/user-attachments/assets/826398bc-ff56-44ad-8b94-89d039a269a0" />


The custom HTML page is displayed.

---

# Task 5: .dockerignore

## .dockerignore

```text
node_modules
.git
*.md
.env
```

## Verify Build Context

```bash
docker build -t ignore-demo:v1 .
```

### Observation

Docker excludes ignored files from the build context, reducing build time and image size.

---

# Task 6: Build Optimization and Cache

## Initial Dockerfile

```dockerfile
FROM ubuntu:latest

COPY . .

RUN apt-get update
```

## Rebuild

```bash
docker build -t cache-demo:v1 .
```

Docker reuses cached layers if unchanged.

---

## Optimized Dockerfile

```dockerfile
FROM ubuntu:latest

RUN apt-get update

COPY . .
```

### Why Layer Order Matters

Docker builds images layer by layer.

When a layer changes:

- That layer and all layers after it must be rebuilt.
- Earlier unchanged layers can be reused from cache.

### Best Practice

Place:

- Package installation
- Dependencies
- System configuration

near the top.

Place:

- Application code
- Frequently changing files

near the bottom.

This maximizes cache reuse and speeds up builds.

---

# Key Commands Learned

```bash
docker build -t image-name:tag .
docker run image-name
docker run -d -p 8080:80 image-name
docker images
docker ps
docker ps -a
docker logs <container-id>
```

---

# Key Takeaways

- Dockerfiles automate image creation.
- Images are built layer by layer.
- CMD provides default commands.
- ENTRYPOINT provides fixed executables.
- .dockerignore reduces build context size.
- Layer ordering improves build performance.
- Nginx can serve static websites from containers.
