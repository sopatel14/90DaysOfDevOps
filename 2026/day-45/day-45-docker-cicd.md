# Day 45 – Docker Build & Push in GitHub Actions

## 🎯 Objective

Build a complete CI/CD pipeline using GitHub Actions that automatically builds a Docker image and publishes it to Docker Hub whenever changes are pushed to the `main` branch.

---

## 📂 Project Used

Instead of creating a sample application, I integrated the workflow into my existing Node.js project.

**GitHub Repository:**

> (https://github.com/sopatel14/nodejs-getting-started)

**Docker Hub Repository:**

> https://hub.docker.com/repositories/sopatel264

---

## ✅ Tasks Completed

* Created a GitHub Actions workflow (`docker-publish.yml`)
* Logged in to Docker Hub securely using GitHub Secrets
* Built the Docker image automatically
* Published the image to Docker Hub
* Tagged the image with `latest`
* Configured the workflow to push images only from the `main` branch
* Verified that feature branches build successfully but do not push images
* Added a GitHub Actions status badge to the project README
* Pulled the image locally and ran the container successfully

---

# GitHub Actions Workflow - docker-publish.yml


```yaml
name: Docker build-push

on:
  push:    

jobs:
  docker-ci:
    runs-on: ubuntu-latest
    


    steps:
      - name: Code Checkout
        uses: actions/checkout@v5


      - name: Docker Login
       # only login if its main branch otherwise its gonna fail
        #if: github.ref == 'refs/heads/main'
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      - name: Docker build and push the image
        uses: docker/build-push-action@v5
        with:
          context: .
          # only push if its main branch otherwise its gonna fail
          push: "${{ github.ref == 'refs/heads/main' }}"
          tags: ${{ vars.DOCKER_USERNAME }}/node-js-new-app:latest

```

---

# Docker Hub Image

https://hub.docker.com/repository/docker/sopatel264/node-js-new-app```

---

# Screenshots

## 1. Successful GitHub Actions Pipeline

<img width="3360" height="1616" alt="image" src="https://github.com/user-attachments/assets/576e0951-a849-466c-ad68-57747984beb0" />


---

## 2. Docker Hub Repository

<img width="3360" height="1698" alt="image" src="https://github.com/user-attachments/assets/e4148ccb-798d-4163-a5a6-fcbdc0bde3f4" />


---

## 3. Workflow Status Badge in README

<img width="1804" height="982" alt="image" src="https://github.com/user-attachments/assets/3412b97b-68a3-45c5-86f6-6c42a582f2d8" />


---

## 4. Docker Desktop / Running Container

<img width="1670" height="666" alt="image" src="https://github.com/user-attachments/assets/e8d15287-456c-45f0-8a76-42af46c7c76f" />


---

## 5. Application Running in Browser

<img width="3342" height="1814" alt="image" src="https://github.com/user-attachments/assets/a7217f49-12e0-4719-95dc-7e4d42a874d3" />

---

# Full Journey: Git Push → Running Container

1. Developer pushes code to the GitHub repository.
2. GitHub Actions workflow is triggered automatically.
3. The runner checks out the source code.
4. GitHub Actions authenticates with Docker Hub using repository secrets.
5. Docker builds the application image.
6. If the branch is `main`, the image is pushed to Docker Hub.
7. On another machine, the image is pulled from Docker Hub.
8. A Docker container is started from the image.
9. The application becomes available on the configured port.

---

# Key Learnings

* Automated Docker image builds using GitHub Actions
* Secure authentication with Docker Hub using GitHub Secrets
* Conditional image publishing based on branch
* Docker image versioning using tags
* End-to-end CI/CD workflow for containerized applications

---

## Result

Successfully implemented an end-to-end Docker CI/CD pipeline that automatically builds, publishes, and deploys container images using GitHub Actions and Docker Hub.
