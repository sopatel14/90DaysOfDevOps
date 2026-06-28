# Day 44 – Secrets, Artifacts & Running Real Tests in CI

## Objective

The goal of Day 44 was to learn how to build more practical and secure GitHub Actions workflows by using **GitHub Secrets**, **Artifacts**, **Caching**, and **Real Test Execution** in a CI pipeline.

---

# Task 1 – GitHub Secrets - .github/workflows/secret-vars.yml

## Workflow File


```yaml
name: Secret Variables

on:
  push:

jobs:
  secret-variable:
    runs-on: ubuntu-latest

    steps:
     - name: Print the secret
       run: |
         echo "The secret is set: true"
         echo "${{ secrets.MY_SECRET_MESSAGE }}"
```

---

## What I Did

* Created a repository secret named:

```
MY_SECRET_MESSAGE
```

* Used the secret inside a GitHub Actions workflow.
* Verified that the workflow detects whether the secret exists.
* Attempted to print the secret directly.

---

## Output


<img width="3356" height="1584" alt="image" src="https://github.com/user-attachments/assets/e5203443-6743-4a9d-9176-8b626564ac0c" />



---

## Observation

GitHub automatically **masks secret values** in workflow logs.

Instead of displaying the actual value, GitHub replaces it with:

```text
***
```

This prevents accidental exposure of sensitive information.

---

## Why You Should Never Print Secrets

Printing secrets in CI logs is a serious security risk because:

* Anyone with access to workflow logs may view them.
* Secrets can be stolen and misused.
* API keys, passwords, and tokens should never appear in logs.
* Even though GitHub masks many secrets, developers should never intentionally print them.

---

# Task 2 – Using Secrets as Environment Variables

---

## Secrets Added

* MY_SECRET_MESSAGE
* DOCKER_USERNAME
* DOCKER_TOKEN

---

## What I Learned

Secrets can be safely passed to workflow steps using environment variables.

This allows applications, scripts, or Docker commands to authenticate without exposing sensitive credentials inside the repository.

---


# Task 3 – Upload Artifacts - .github/workflows/upload-artifacts.yml

## Workflow File

```yaml
name: Artifacts 

on:
  push:

jobs:
  artifact-upload:
    runs-on: ubuntu-latest

    steps:
      - name: Generate a file
        run: echo "Hello World, This is Test report" > test-report.txt

      - name: Upload the file
        uses: actions/upload-artifact@v4
        with:
          name: test-report-artifact
          path: test-report.txt

  artifact-download:
    runs-on: ubuntu-latest
    needs: artifact-upload
    steps:
        - name: Download Artifacts
          uses: actions/download-artifact@v4
          with:
            name: test-report-artifact

        - name: View file contents
          run: cat test-report.txt
```

---

## What I Did

The workflow generated a report file during execution.


The file was uploaded using:

* actions/upload-artifact

Artifacts allow workflow-generated files to be downloaded even after the workflow has completed.

---

## Artifact Download

## Output



<img width="3338" height="1686" alt="image" src="https://github.com/user-attachments/assets/bb611ce0-3cb9-447a-97ca-aedd0aa6121f" />



---

## Verification

✔ Artifact successfully uploaded.

✔ Artifact downloaded from the Actions tab.

---

# Task 4 – Download Artifacts Between Jobs

## Workflow File - .github/workflows/upload-artifacts.yml


```yaml
name: Artifacts 

on:
  push:

jobs:
  artifact-upload:
    runs-on: ubuntu-latest

    steps:
      - name: Generate a file
        run: echo "Hello World, This is Test report" > test-report.txt

      - name: Upload the file
        uses: actions/upload-artifact@v4
        with:
          name: test-report-artifact
          path: test-report.txt

  artifact-download:
    runs-on: ubuntu-latest
    needs: artifact-upload
    steps:
        - name: Download Artifacts
          uses: actions/download-artifact@v4
          with:
            name: test-report-artifact

        - name: View file contents
          run: cat test-report.txt

```

---

## Workflow Design

### Job 1

* Generates a file
* Uploads it as an artifact

↓

### Job 2

* Downloads the artifact
* Reads its contents
* Prints the file

---

## Output


<img width="3360" height="1188" alt="image" src="https://github.com/user-attachments/assets/eabc6c12-0740-4295-87a6-5549f6e96dfb" />

<img width="3360" height="1398" alt="image" src="https://github.com/user-attachments/assets/36d56c54-300e-4117-b0ca-d0d7fb608743" />



---

## When Are Artifacts Useful?

Artifacts are commonly used to transfer files between jobs without rebuilding them.

Examples include:

* Test reports
* Build logs
* Docker images
* Compiled binaries
* Coverage reports
* Deployment packages

They also help preserve important workflow outputs for later inspection.

---

# Task 5 – Running Real Tests in CI

## Workflow File - .github/workflows/workflow-scan.yml

```yaml
name: Scan script

on:
  push:
      

jobs:
  scan-script:
    runs-on: ubuntu-latest
    steps:
      - name: Code Checkout
        uses: actions/checkout@v7

      - name: Caching step
        uses: actions/cache@v4
        with:
          path: my-cache-folder
          key: script-deps-v1

      - name: Make the script files executable
        run: chmod +x system_info.sh

      - name: Run the script
        run: ./system_info.sh

```

---

## Script Used - system-info.sh


```yaml
#!/bin/bash

set -euo pipefail

hostname_info() {
    echo "===== HOSTNAME & OS ====="
    hostname
    cat /etc/os-release | grep PRETTY_NAME
}

uptime_info() {
    echo
    echo "===== UPTIME ====="
    uptime
}

disk_info() {
    echo
    echo "===== DISK USAGE ====="
    du -sh /* 2>/dev/null | sort -hr | head -n 5 || true
}

memory_info() {
    echo
    echo "===== MEMORY USAGE ====="
    free -h
}

cpu_info() {
    echo
    echo "===== TOP CPU PROCESSES ====="
    ps aux --sort=-%cpu | head -6
}

main() {
    hostname_info
    uptime_info
    disk_info
    memory_info
    cpu_info
}

main


# Check if the cache folder is empty
if [ ! -d "my-cache-folder" ] || [ -z "$(ls -A my-cache-folder)" ]; then
  echo "Cache Miss! Downloading files..."
  mkdir -p my-cache-folder
  
  # This simulates a download by creating a fake 10MB file
  dd if=/dev/urandom of=my-cache-folder/fake-deps.bin bs=1M count=10
else
  echo "Cache Hit! Skipping download."
fi
```

---

## What I Did

The workflow:

1. Checked out the repository.
2. Installed required dependencies.
3. Executed the script.
4. Failed automatically when the script exited with a non-zero exit code.
5. Passed successfully after fixing the issue.

---

## Passing Workflow

## Output



<img width="3360" height="1680" alt="image" src="https://github.com/user-attachments/assets/491c5498-536d-4eb0-bfa7-f22bc14eb7d5" />


---

## What I Learned

GitHub Actions automatically marks a workflow as failed whenever any command exits with a non-zero status code.

This ensures broken code is detected before deployment.

---

# Task 6 – Dependency Caching

## Workflow File

```yaml
name: Scan script

on:
  push:
      

jobs:
  scan-script:
    runs-on: ubuntu-latest
    steps:
      - name: Code Checkout
        uses: actions/checkout@v7

      - name: Caching step
        uses: actions/cache@v4
        with:
          path: my-cache-folder
          key: script-deps-v1

      - name: Make the script files executable
        run: chmod +x system_info.sh

      - name: Run the script
        run: ./system_info.sh
```

---

## What I Did

Configured dependency caching using:

* actions/cache

The workflow was executed multiple times.

The first run downloaded and installed dependencies.

Subsequent runs restored dependencies from cache, reducing workflow execution time.

---

## Output

📸 **Screenshot Placeholder**


<img width="3334" height="1646" alt="image" src="https://github.com/user-attachments/assets/8c44ba34-689b-45cc-87b4-2acdc12937ed" />

<img width="3354" height="1722" alt="image" src="https://github.com/user-attachments/assets/cd9438f8-5b09-4934-9740-b22883d731b3" />




---

## What Is Being Cached?

Typical cached files include:

* Python packages (`pip`)
* Node.js dependencies (`node_modules` or npm cache)
* Maven packages
* Gradle cache
* Composer dependencies
* Other package manager caches

---

## Where Is the Cache Stored?

GitHub stores workflow caches on its infrastructure.

Each cache is associated with:

* Repository
* Branch
* Cache key

Future workflow runs restore matching caches automatically, improving build performance.

---

# Key Learnings

* Learned how GitHub Secrets securely store sensitive information.
* Understood why secrets should never be exposed in CI logs.
* Used environment variables to securely access secrets.
* Uploaded and downloaded artifacts between workflow jobs.
* Executed real test scripts inside GitHub Actions.
* Observed workflow failures and successful reruns.
* Used dependency caching to improve workflow speed.
* Gained a deeper understanding of secure and efficient CI pipelines.

---

# Conclusion

Day 44 focused on building more production-ready GitHub Actions workflows. I learned how to securely manage sensitive data using GitHub Secrets, preserve workflow outputs with Artifacts, share files between jobs, execute automated tests, and speed up builds using dependency caching. These features are essential for creating reliable, secure, and efficient CI pipelines used in real-world DevOps environments.
