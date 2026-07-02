# Day 49 – DevSecOps Implementation

https://github.com/sopatel14/flask-app-ecs

## 📌 What is DevSecOps?

DevSecOps is the practice of integrating security into every stage of the CI/CD pipeline instead of treating it as a separate phase. By automating security checks during development, testing, and deployment, vulnerabilities can be identified and fixed much earlier, reducing security risks and improving software quality.

---

# Task 1 – Scan Docker Images with Trivy

## Objective

Integrate Trivy into the GitHub Actions pipeline to scan Docker images for known vulnerabilities before pushing them to Docker Hub.

---

## Workflow Implementation

```yaml
- name: Scan Docker Image for Vulnerabilities
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: ${{ secrets.DOCKER_USERNAME }}/flask-app-ecs:latest
    format: table
    exit-code: 1
    severity: CRITICAL,HIGH
```

---

## How It Works

- Builds the Docker image.
- Scans the image for known CVEs.
- Displays the scan result in a readable table.
- Fails the workflow if any **HIGH** or **CRITICAL** vulnerabilities are found.
- Prevents vulnerable images from being pushed.

---

## Initial Scan Result

**Base Image Used**

```dockerfile
FROM python:3.14-slim
```

### Vulnerabilities Found

| Severity | Count |
|----------|------:|
| CRITICAL | 2 |
| HIGH | 10 |

Example CVEs detected:

- CVE-2026-42496
- CVE-2026-41992
- CVE-2026-54369
- CVE-2026-54371
- CVE-2025-69720

---

## Fix Applied

The base image was changed from Debian Slim to Alpine Linux.

```dockerfile
FROM python:3.14-alpine

RUN apk add --no-cache \
    gcc \
    musl-dev \
    libffi-dev \
    && apk upgrade --no-cache

WORKDIR /app

COPY . .

RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 80

CMD ["python","run.py"]
```

---

## Final Scan Result

```
Total Vulnerabilities: 0

UNKNOWN: 0
LOW: 0
MEDIUM: 0
HIGH: 0
CRITICAL: 0
```

---

### 📷 Screenshot
<img width="1680" height="835" alt="image" src="https://github.com/user-attachments/assets/a4e5712a-098d-49aa-821f-6180bdc88c68" />

<img width="3342" height="1694" alt="image" src="https://github.com/user-attachments/assets/1b18e9e9-f943-42df-90e8-4c8d5ff68a8f" />


---

# Task 2 – Enable GitHub Secret Scanning

## Objective

Enable GitHub's built-in Secret Scanning and Push Protection to prevent credentials from being committed.

---

## Steps Performed

- Repository → Settings
- Code Security and Analysis
- Enabled:
  - ✅ Secret Scanning
  - ✅ Push Protection

---

## How It Works

### Secret Scanning

- Detects secrets already committed.
- Alerts repository administrators.
- Creates security alerts.

### Push Protection

- Detects secrets before a push completes.
- Blocks the push.
- Prevents accidental credential leaks.

---

## Benefits

- Automatic secret detection
- Real-time protection
- Prevents leaked AWS keys
- Integrated directly into GitHub

---

### 📷 Screenshot

<img width="1102" height="184" alt="image" src="https://github.com/user-attachments/assets/0a488c4a-5e94-4dd8-a74e-d7f0dfbfa827" />


---

# Task 3 – Dependency Review

## Objective

Automatically scan newly added dependencies in Pull Requests for known vulnerabilities.

---

## Workflow

```yaml
dependency-review:
  runs-on: ubuntu-latest

  steps:
    - uses: actions/checkout@v4

    - uses: actions/dependency-review-action@v4
      with:
        fail-on-severity: critical
```

---

## How It Works

- Runs on Pull Requests.
- Compares dependency changes.
- Checks GitHub Advisory Database.
- Fails if critical vulnerabilities exist.
- Prevents unsafe dependencies from being merged.

---

### 📷 Screenshot


<img width="3360" height="1750" alt="image" src="https://github.com/user-attachments/assets/30b84dde-b820-4732-ac9c-eb02c416f1db" />

---

# Task 4 – Restrict Workflow Permissions

## Objective

Follow the Principle of Least Privilege by giving workflows only the permissions they actually need.

---

## Main Pipeline

```yaml
permissions:
  contents: read
```

---

## Docker Job

```yaml
permissions:
  contents: read
  packages: write
```

---

## PR Pipeline

```yaml
permissions:
  contents: read
  pull-requests: write
```

---

## Why Limit Permissions?

If a GitHub Action becomes compromised, limited permissions help prevent:

- Source code modification
- Unauthorized deployments
- Secret theft
- Package tampering
- Infrastructure abuse

---

# Task 5 – DevSecOps Pipeline

```text
                     Pull Request

      Build & Test
            │
            ▼
     Dependency Review
            │
            ▼
      PR Passes / Fails


                     Main Branch

      Build & Test
            │
            ▼
       Docker Build
            │
            ▼
        Trivy Scan
            │
      (Fail if HIGH/CRITICAL)
            │
            ▼
       Docker Push
            │
            ▼
      Deploy to AWS ECS


          Always Running in Background

       GitHub Secret Scanning
                 │
                 ▼
          Push Protection
                 │
                 ▼
      Prevent Secret Leakage
```

---

# Key Learnings

## DevSecOps

Security should be part of the CI/CD pipeline rather than a final step before production.

---

## Security Checks Added

| Security Check | Trigger | Purpose |
|---------------|---------|---------|
| Trivy Scan | Main Branch | Docker image vulnerability scanning |
| Dependency Review | Pull Request | Detect vulnerable dependencies |
| Secret Scanning | Always | Detect leaked credentials |
| Push Protection | Push | Block commits containing secrets |

---

## Security Principles Applied

- Shift Security Left
- Continuous Security
- Automated Vulnerability Detection
- Principle of Least Privilege
- Secure by Default

---

# Conclusion

Day 49 focused on integrating security directly into the CI/CD pipeline.

The project now automatically:

- Scans Docker images
- Reviews dependency vulnerabilities
- Detects leaked secrets
- Blocks insecure code
- Restricts GitHub Actions permissions

These additions make the deployment pipeline significantly more secure and align it with modern DevSecOps best practices.


---

Dockerfile

day-49-devsecops.md
```
