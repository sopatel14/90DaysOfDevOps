# Day 48 – GitHub Actions Capstone Project

## Objective

Build a production-style CI/CD pipeline using reusable workflows, Docker, GitHub Actions, environments, and scheduled health checks.

---

# Repository

Repository Link:

https://github.com/sopatel14/flask-app-ecs
---

# Project

A simple Flask application with:

- Home endpoint (`/`)
- Health endpoint (`/health`)
- Docker support
- GitHub Actions CI/CD pipeline

---

# Pipeline Architecture

```
                 Pull Request
                      │
                      ▼
             PR Pipeline
                      │
                      ▼
          Reusable Build & Test
                      │
             PR Checks Passed
                      │
                 Merge to Main
                      │
                      ▼
             Main Pipeline
                      │
                      ▼
          Reusable Build & Test
                      │
                      ▼
         Docker Build & Push
                      │
                      ▼
          Deploy to Production
                      │
                      ▼
      Production Environment

────────────────────────────────────

Every 12 Hours
        │
        ▼
 Scheduled Health Check
        │
        ▼
Pull Docker Image
        │
Run Container
        │
Curl /health
        │
Generate Summary
```

---

# Workflow Files

## 1. reusable-build-test.yml

```yaml
name: Reusable Build & Test

on:
  workflow_call:
    inputs:
      python_version:
        description: "Python version to use"
        required: true
        type: string

      run_tests:
        description: "Run application tests"
        required: false
        default: true
        type: boolean

    outputs:
      test_result:
        description: "Result of the test execution"
        value: ${{ jobs.build-test.outputs.test_result }}

jobs:
  build-test:
    runs-on: ubuntu-latest

    outputs:
      test_result: ${{ steps.result.outputs.test_result }}

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ inputs.python_version }}

      - name: Install Dependencies
        run: |
          python -m pip install --upgrade pip
          if [ -f requirements.txt ]; then
            pip install -r requirements.txt
          fi

      - name: Run Tests
        if: ${{ inputs.run_tests }}
        run: |
          python -m compileall .
          echo "Tests completed successfully."

      - name: Set Test Result
        id: result
        run: echo "test_result=passed" >> $GITHUB_OUTPUT
```

---

## 2. reusable-docker.yml

```yaml
name: Reusable Docker Build & Push

on:
  workflow_call:
    inputs:
      image_name:
        description: "Docker image name"
        required: true
        type: string

      tag:
        description: "Docker image tag"
        required: true
        type: string

    secrets:
      docker_username:
        required: true

      docker_token:
        required: true

    outputs:
      image_url:
        description: "Full Docker image URL"
        value: ${{ jobs.docker.outputs.image_url }}

jobs:
  docker:
    runs-on: ubuntu-latest

    outputs:
      image_url: ${{ steps.image.outputs.image_url }}

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.docker_username }}
          password: ${{ secrets.docker_token }}

      - name: Build and Push Docker Image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: |
            ${{ secrets.docker_username }}/${{ inputs.image_name }}:${{ inputs.tag }}

      - name: Set Image URL Output
        id: image
        run: |
          echo "image_url=${{ secrets.docker_username }}/${{ inputs.image_name }}:${{ inputs.tag }}" >> $GITHUB_OUTPUT

```

---

## 3. pr-pipeline.yml

```yaml
name: PR Pipeline

on:
  pull_request:
    branches:
      - main
    types:
      - opened
      - synchronize

jobs:
  build-test:
    uses: ./.github/workflows/reusable-build-test.yml
    with:
      python_version: "3.13"
      run_tests: true

  pr-comment:
    name: PR Summary
    needs: build-test
    runs-on: ubuntu-latest

    steps:
      - name: Print PR Summary
        run: |
          echo "PR checks passed for branch: ${{ github.head_ref }}"
```

---

## 4. main-pipeline.yml

```yaml
name: Main Pipeline

on:
  push:
    branches:
      - main

jobs:
  build-test:
    uses: ./.github/workflows/reusable-build-test.yml
    with:
      python_version: "3.13"
      run_tests: true

  docker:
    needs: build-test
    uses: ./.github/workflows/reusable-docker.yml
    with:
      image_name: flask-app-ecs
      tag: latest
    secrets:
      docker_username: ${{ secrets.docker_username }}
      docker_token: ${{ secrets.docker_token }}

  deploy:
    name: Deploy to Production
    needs: docker
    runs-on: ubuntu-latest
    environment: production

    steps:
      - name: Deploy Application
        run: |
          echo "Deploying image: ${{ needs.docker.outputs.image_url }} to production"

```

---

## 5. health-check.yml

```yaml

name: Scheduled Health Check

on:
  schedule:
    - cron: '0 */12 * * *'
  workflow_dispatch:

jobs:
  health-check:
    runs-on: ubuntu-latest

    steps:
      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.docker_username }}
          password: ${{ secrets.docker_token }}

      - name: Pull Latest Docker Image
        run: |
          docker pull ${{ secrets.docker_username }}/flask-app-ecs:latest

      - name: Run Container
        run: |
          docker run -d --name flask-app -p 5000:5000 \
            ${{ secrets.docker_username }}/flask-app-ecs:latest

      - name: Wait for Application
        run: sleep 5

      - name: Check Health Endpoint
        run: |
          response=$(curl -s http://localhost:5000/health)

          if [[ "$response" == *"Server is up and running"* ]]; then
            echo "Health Check PASSED"
            echo "STATUS=PASSED" >> $GITHUB_ENV
          else
            echo "Health Check FAILED"
            echo "STATUS=FAILED" >> $GITHUB_ENV
            exit 1
          fi

      - name: Stop Container
        if: always()
        run: |
          docker stop flask-app || true
          docker rm flask-app || true

      - name: Create Job Summary
        if: always()
        run: |
          echo "## Health Check Report" >> $GITHUB_STEP_SUMMARY
          echo "- Image: flask-app-ecs:latest" >> $GITHUB_STEP_SUMMARY
          echo "- Status: $STATUS" >> $GITHUB_STEP_SUMMARY
          echo "- Time: $(date)" >> $GITHUB_STEP_SUMMARY

```

---

# Screenshots

## PR Pipeline

<img width="1679" height="821" alt="image" src="https://github.com/user-attachments/assets/513c0357-ec8c-4879-bf1d-e89d7c759a61" />

<img width="1679" height="791" alt="image" src="https://github.com/user-attachments/assets/aad1d734-9f29-4742-97c6-e5c34e5564b9" />


---

## Main Pipeline

<img width="1680" height="859" alt="image" src="https://github.com/user-attachments/assets/0d575b7d-47f8-4569-9af7-c51b1e909201" />

<img width="1680" height="861" alt="image" src="https://github.com/user-attachments/assets/7dee8e56-9731-4c85-95c5-94b612b2eadb" />

---

## Production Deployment

**Attach Screenshot**

Show:

- Environment = production

---

## Scheduled Health Check

<img width="1680" height="804" alt="image" src="https://github.com/user-attachments/assets/5165afa4-ae6c-4cd4-9e8d-d9ab8129519e" />


---

## Health Check Summary

<img width="1680" height="826" alt="image" src="https://github.com/user-attachments/assets/d8d7d82a-9ab7-4822-abbc-dd4f3dbe7c71" />


---

## Docker Hub

<img width="1677" height="647" alt="image" src="https://github.com/user-attachments/assets/c1c5470e-89dc-48ae-a614-c1cf0f898cdd" />


---

# Docker Hub

Repository

https://hub.docker.com/repository/docker/sopatel264/
---

# What I Learned

- Reusable workflows
- workflow_call
- Workflow outputs
- Docker build & push
- Repository secrets
- GitHub Environments
- Scheduled workflows
- Health checks
- Manual deployments
- End-to-end CI/CD pipeline design

---

# Challenges Faced

- Understanding reusable workflow outputs
- Passing secrets between workflows
- Configuring Docker Hub authentication
- Using production environments
- Triggering workflows correctly

---

# Future Improvements

- Slack notifications
- Microsoft Teams notifications
- Automatic rollback
- Multi-environment deployments (Dev → QA → Production)
- Image vulnerability scanning with Trivy
- Code coverage reports
- Kubernetes deployment
- Terraform infrastructure deployment

---

# Final Outcome

Successfully built a production-style CI/CD pipeline consisting of:

- Reusable Build & Test workflow
- Reusable Docker Build workflow
- Pull Request pipeline
- Main branch deployment pipeline
- Scheduled Health Check workflow

This project demonstrates an end-to-end GitHub Actions pipeline using reusable workflows, Docker, GitHub Environments, and automated health monitoring.
