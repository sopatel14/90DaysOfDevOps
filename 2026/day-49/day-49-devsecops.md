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

## 📝 Notes from All Tasks (As Required)

### Task 1 Notes

**Q: What CVEs (if any) were found? What base image are you using?**

**A:** The scan found 12 vulnerabilities (2 CRITICAL, 10 HIGH) in the `python:3.14-slim` base image:

**CRITICAL Vulnerabilities (2):**
- **CVE-2026-42496** - perl-archive-tar: Path traversal via crafted symlinks allowing arbitrary file access
- **CVE-2026-8376** - Perl: Heap buffer overflow when compiling regular expressions on 32-bit builds

**HIGH Vulnerabilities (10):**
- **CVE-2026-41992** - GNU gzip buffer overflow vulnerability in LZH decompression
- **CVE-2026-54369** - acl: Symlink traversal privilege escalation via libacl functions
- **CVE-2026-54371** - attr: Symlink traversal privilege escalation via getfattr and setfattr
- **CVE-2025-69720** - ncurses: Buffer overflow vulnerability leading to arbitrary code execution
- **CVE-2026-42497** - perl-Archive-Tar: Arbitrary file modification via crafted hardlinks
- **CVE-2026-48962** - perl-IO-Compress: Arbitrary code execution via attacker-controlled output glob
- **CVE-2026-9538** - Archive::Tar: Memory exhaustion vulnerability

The base image was `python:3.14-slim` (Debian-based). After switching to Alpine Linux (`python:3.14-alpine`), all vulnerabilities were resolved (Total: 0).

---

### Task 2 Notes

**Q: What is the difference between secret scanning and push protection?**

**A:** 
- **Secret Scanning**: Detects secrets that are already in your repository and alerts you. It's a reactive measure - it finds secrets after they've been committed and notifies administrators.
- **Push Protection**: Blocks the commit/push if a secret is detected. It's a proactive measure - it prevents secrets from ever being committed in the first place.

**Q: What happens if GitHub detects a leaked AWS key in your repo?**

**A:** 
1. GitHub secret scanning immediately alerts repository administrators
2. Push protection blocks the commit from being pushed (if enabled)
3. AWS automatically invalidates the exposed credentials
4. A security advisory is created in the repository
5. The commit is flagged and must be fixed before merging

---

### Task 3 Notes

**Q: Does the dependency review show up as a check on your PR?**

**A:** Yes, the dependency review appears as a separate check on the PR page alongside build-test and pr-comment checks. It runs automatically when a PR is opened or updated.

**Q: What happens if a dependency has a critical CVE?**

**A:** The dependency-review job fails and the PR cannot be merged. The pipeline blocks the PR until the vulnerable dependency is removed or updated to a secure version.

---

### Task 4 Notes

**Q: Why is it a good practice to limit workflow permissions? What could go wrong if a compromised action has write access to your repo?**

**A:** Limiting workflow permissions follows the **principle of least privilege**. If a compromised action has write access, the following could happen:

| Threat | Impact |
|--------|--------|
| Source code modification | Attackers could inject malicious code |
| Production deployment | Malicious code could be pushed to production |
| Secret theft | Other secrets and credentials could be stolen |
| Infrastructure abuse | GitHub's infrastructure could be used for attacks |
| Supply chain attack | The compromised action could affect other repos or users |
| Data exfiltration | Sensitive data could be stolen |
| Backdoor injection | Permanent access could be established |

---

### Summary of All Notes

| Task | Required Notes | Answer |
|------|----------------|--------|
| Task 1 | CVEs found, base image used | 12 vulnerabilities (2 CRITICAL, 10 HIGH) in python:3.14-slim; fixed by switching to Alpine |
| Task 2 | Difference between secret scanning and push protection | Secret scanning finds committed secrets; push protection blocks them from being committed |
| Task 2 | What happens if AWS key leaked | GitHub alerts, AWS revokes credentials, security advisory created |
| Task 3 | Does dependency review show on PR | Yes, appears as a check on the PR page |
| Task 4 | Why limit permissions + what could go wrong | Prevents compromised actions from modifying code, stealing secrets, or causing supply chain attacks |

---
