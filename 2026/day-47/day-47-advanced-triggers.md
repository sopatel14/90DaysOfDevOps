# Day 47 – Advanced Triggers: PR Events, Cron Schedules & Event-Driven Pipelines

## 📌 Objective

Today I explored advanced GitHub Actions triggers used in real-world CI/CD pipelines. I learned how to trigger workflows based on Pull Request lifecycle events, scheduled cron jobs, path filters, workflow chaining, and external events.

---

# Task 1 – Pull Request Lifecycle Events

## 🎯 Goal

Create a workflow that reacts to different Pull Request activities.

### Triggered Events

- opened
- synchronize
- reopened
- closed

### Workflow File - .github/workflows/pr-lifecycle.yml


```yaml
# Create a workflow that runs during different stages of a Pull Request lifecycle.


name: PR lifecycle

on:
  pull_request:
    types: [opened, synchronize, reopened, closed]

jobs:
  pr-info:
    runs-on: ubuntu-latest

    steps:
      - name: Event type fired
        run: echo "The event action is ${{ github.event.action }}"

      - name: Print the title of the PR
        run: echo "The Pull Request title is ${{ github.event.pull_request.title }}"

      - name: Print the PR author
        run: echo "The author of this PR is ${{ github.event.pull_request.user.login }}"

      - name: Print Source and Target Branches
        run: |
          echo "Source Branch: ${{ github.head_ref }}"
          echo "Target Branch: ${{ github.base_ref }}"

      - name: Only Runs when PR is merged
        if: github.event.action == 'closed' && github.event.pull_request.merged == true
        run: |
          echo "The PR was successfully merged"

      - name: Only Runs when PR is closed without merging
        if: github.event.action == 'closed' && github.event.pull_request.merged == false
        run: |
          echo "The PR was closed without merging."

```

### What I Learned

- `github.event.action` tells which PR activity triggered the workflow.
- PR metadata such as:
  - Title
  - Author
  - Source branch
  - Target branch
- Detect whether a PR was merged using:

```yaml
github.event.pull_request.merged == true
```

---

## Output
<img width="3350" height="1732" alt="image" src="https://github.com/user-attachments/assets/014d884c-6eb8-4b13-9b60-271796eec696" />

<img width="3352" height="1698" alt="image" src="https://github.com/user-attachments/assets/5943377a-135e-428a-8d58-f3236ed08ae0" />

<img width="3338" height="1678" alt="image" src="https://github.com/user-attachments/assets/ff996d88-e2c1-45b4-b879-31c73aba9575" />

<img width="3358" height="1690" alt="image" src="https://github.com/user-attachments/assets/8f3637c8-4493-4bdb-828d-9c3686e037c4" />



---

# Task 2 – Pull Request Validation

## 🎯 Goal

Automatically validate Pull Requests before merging.

### Workflow File - .github/workflows/pr-checks.yml


```yaml

name: PR Validation Workflow

on:
  pull_request:
    branches: [main]

jobs:
  file-size-check:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code 
        uses: actions/checkout@v5

      - name: Fails if File larger than 1 MB
        run: |
          if find . -type f -size +1M | grep -q .; then
            echo "Large files found:"
            find . -type f -size +1M -exec ls -lh {} \;
            echo "::error::Files larger than 1MB found in PR"
            exit 1
          else
            echo "All files are under 1MB"
          fi

  branch-name-check:
    runs-on: ubuntu-latest

    steps:
      - name: Validate branch name pattern
        run: |
          BRANCH="${{ github.head_ref }}"
          echo "Checking branch: $BRANCH"
          
          if [[ "$BRANCH" =~ ^(feature|fix|docs)/ ]]; then
            echo "Branch name '$BRANCH' follows the naming convention"
          else
            echo "Branch name '$BRANCH' is invalid"
            echo "Branch names must start with: feature/, fix/, or docs/"
            echo "::error::Invalid branch name format"
            exit 1
          fi

  pr-body-check:
    runs-on: ubuntu-latest

    steps:
      - name: Read PR Body Check
        run: |
          PR_BODY="${{ github.event.pull_request.body }}"
          echo "Checking PR description..."
          
          if [ -z "$PR_BODY" ]; then
            echo "PR description is empty"
            echo "::warning::Please add a description to your PR"
          else
            echo "PR description is not empty"
            echo "Description preview: ${PR_BODY:0:100}..."
          fi
```

### Implemented Checks

### ✅ File Size Validation

- Fail workflow if any file exceeds **1 MB**

### ✅ Branch Naming Validation

Allowed prefixes:

- feature/
- fix/
- docs/

### ✅ Pull Request Description Check

Warn if the PR body is empty.

---

## Output


<img width="1679" height="832" alt="image" src="https://github.com/user-attachments/assets/035baa0e-c3f2-41b4-bc2f-81731f9eca3b" />


---

# Task 3 – Scheduled Workflows (Cron)

## 🎯 Goal

Run workflows automatically without manual intervention.

### Workflow File - .github/workflows/scheduled-tasks.yml

```yaml

name: Scheduled Tasks

on:
  schedule:
    - cron: '30 2 * * 1'    # Every Monday at 2:30 AM UTC
    - cron: '0 */6 * * *'   # Every 6 hours
  workflow_dispatch:        # Manual trigger for testing

jobs:
  scheduled-job:
    runs-on: ubuntu-latest
    
    steps:
      - name: Print schedule trigger
        run: |
          echo "Triggered by schedule: ${{ github.event.schedule }}"
          if [ "${{ github.event.schedule }}" == "30 2 * * 1" ]; then
            echo "This is the weekly Monday run"
          elif [ "${{ github.event.schedule }}" == "0 */6 * * *" ]; then
            echo "This is the 6-hourly run"
          else
            echo "This was triggered manually (workflow_dispatch)"
          fi
          
      - name: Health check
        run: |
          echo "Performing health check..."
          RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" https://github.com)
          echo "Response code: $RESPONSE"
          
          if [ $RESPONSE -eq 200 ]; then
            echo "Health check passed"
          else
            echo "Health check failed with status: $RESPONSE"
            exit 1
          fi
```

### Cron Jobs Used

| Schedule | Cron |
|-----------|------|
| Every Monday 2:30 UTC | `30 2 * * 1` |
| Every 6 Hours | `0 */6 * * *` |

Also enabled:

```yaml
workflow_dispatch
```

for manual testing.

---

### Health Check

The workflow performs a simple health check using:

```bash
curl
```

to verify GitHub is reachable.

---

## Output

<img width="3360" height="1690" alt="image" src="https://github.com/user-attachments/assets/5824d041-01b9-46bd-9678-2a8b2814baca" />


---

# Task 4 – Path & Branch Filters

## 🎯 Goal

Run workflows only when relevant files change.

### Workflow Files - .github/workflows/smart-triggers.yml & .github/workflows/smart-triggers-ignore.yml

```yaml

name: Smart Triggers

on:
  push:
    branches:
      - main
      - release/*
    paths:
      - '*.py'
      - 'app.py'

jobs:
  smart-job:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v5
      
      - name: Show trigger information
        run: |
          echo "========================================"
          echo "Workflow triggered by paths filter"
          echo "Branch: ${{ github.ref_name }}"
          echo "Commit: ${{ github.sha }}"
          echo "Commit message: ${{ github.event.head_commit.message }}"
          echo "========================================"
      
      - name: Show changed files
        run: |
          echo "Files changed in this push:"
          echo "${{ github.event.head_commit.modified }}"
          echo "========================================"
```

```yaml

name: Smart Triggers - Ignore

on:
  push:
    branches:
      - main
      - release/*
    paths-ignore:
      - '*.md'
      - 'docs/**'

jobs:
  smart-job:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v5
      
      - name: Show trigger information
        run: |
          echo "========================================"
          echo "Workflow triggered by paths-ignore filter"
          echo "Branch: ${{ github.ref_name }}"
          echo "Commit message: ${{ github.event.head_commit.message }}"
          echo "========================================"
      
      - name: Show changed files
        run: |
          echo "Files changed in this push:"
          git diff --name-only HEAD~1
          echo "========================================"
```

### Learned

### paths

Run workflow only when selected files change.

Example:

```yaml
paths:
  - "*.py"
  - app.py
```

### paths-ignore

Skip workflow for documentation changes.

Example:

```yaml
paths-ignore:
  - "*.md"
  - docs/**
```

---

## Output

<img width="3360" height="1656" alt="image" src="https://github.com/user-attachments/assets/9bebd1da-9086-41fc-81a4-20c00c3c368a" />


---

# Task 5 – Workflow Chaining

## 🎯 Goal

Run deployment only after tests complete successfully.

### Workflow Files - .github/workflows/tests.yml & .github/workflows/deploy-after-tests.yml


```yaml

name: Run Tests

on:
  push:
    branches:
      - main
      - release/*

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v5
      
      - name: Run tests
        run: |
          echo "========================================"
          echo "Running tests..."
          echo "========================================"
          
          echo "Test 1: Checking app.py exists"
          if [ -f "app.py" ]; then
            echo "PASS: app.py found"
          else
            echo "FAIL: app.py not found"
            exit 1
          fi
          
          echo "Test 2: Checking Python syntax"
          if command -v python3 &> /dev/null; then
            echo "PASS: Python is installed"
          else
            echo "WARNING: Python not found (skipping syntax check)"
          fi
          
          echo "Test 3: All tests passed!"
          echo "========================================"

```

```yaml

name: Deploy After Tests

on:
  workflow_run:
    workflows: ["Run Tests"]
    types:
      - completed

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v5
      
      - name: Check if tests passed
        run: |
          echo "========================================"
          echo "Checking test workflow conclusion..."
          echo "========================================"
          
          CONCLUSION="${{ github.event.workflow_run.conclusion }}"
          echo "Test workflow conclusion: $CONCLUSION"
          
          if [ "$CONCLUSION" == "success" ]; then
            echo "Tests passed. Proceeding with deployment..."
          else
            echo "Tests failed. Stopping deployment..."
            echo "::error::Deployment cancelled due to test failures"
            exit 1
          fi
      
      - name: Deploy application
        run: |
          echo "========================================"
          echo "Starting deployment..."
          echo "========================================"
          
          echo "Step 1: Building application..."
          echo "Step 2: Deploying to server..."
          echo "Step 3: Verifying deployment..."
          
          echo "Deployment complete!"
          echo "========================================"
```

### Learned

`workflow_run`

- waits for another workflow
- checks its conclusion
- deploys only if tests pass

Unlike:

`workflow_call`

which is used for reusable workflows.

---

## Output

<img width="3354" height="1606" alt="image" src="https://github.com/user-attachments/assets/d07e0c81-aed1-4a97-b27a-ad09850a8c8e" />

<img width="831" height="55" alt="image" src="https://github.com/user-attachments/assets/acede89e-3f0a-439d-8667-70f1ce54b0e9" />



---

# Task 6 – External Event Trigger

## 🎯 Goal

Trigger GitHub Actions from an external system.

### Workflow File - .github/workflows/external-trigger.yml


```yaml

name: External Trigger

on:
  repository_dispatch:
    types:
      - deploy-request

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v5
      
      - name: Print event information
        run: |
          echo "========================================"
          echo "Repository Dispatch Triggered"
          echo "========================================"
          echo "Event type: ${{ github.event.action }}"
          echo "Client payload: ${{ github.event.client_payload }}"
          echo "========================================"
      
      - name: Extract environment from payload
        run: |
          echo "========================================"
          echo "Extracting deployment information..."
          echo "========================================"
          
          ENVIRONMENT="${{ github.event.client_payload.environment }}"
          echo "Environment: $ENVIRONMENT"
          
          if [ -n "$ENVIRONMENT" ]; then
            echo "Deploying to: $ENVIRONMENT"
            
            if [ "$ENVIRONMENT" == "production" ]; then
              echo "Production deployment - proceeding with caution"
            elif [ "$ENVIRONMENT" == "staging" ]; then
              echo "Staging deployment - safe to proceed"
            else
              echo "Unknown environment: $ENVIRONMENT"
            fi
          else
            echo "No environment specified in payload"
          fi
          echo "========================================"
      
      - name: Simulate deployment
        run: |
          echo "========================================"
          echo "Starting deployment process..."
          echo "========================================"
          echo "Step 1: Building application"
          echo "Step 2: Running pre-deployment checks"
          echo "Step 3: Deploying application"
          echo "Step 4: Post-deployment verification"
          echo "Deployment completed successfully"
          echo "========================================"
```

### Trigger

```yaml
repository_dispatch
```

### Payload Example

```json
{
  "environment": "production"
}
```

### Learned

This trigger is useful for:

- Slack deployment commands
- Jenkins pipelines
- External schedulers
- Monitoring systems
- Third-party integrations

---

## Output

<img width="831" height="55" alt="image" src="https://github.com/user-attachments/assets/ad5b3c56-8c9a-4930-9259-9284f7957209" />

<img width="3360" height="1670" alt="image" src="https://github.com/user-attachments/assets/af3b05ee-5b80-4b76-a4ed-9fbecad7bf00" />


---

# Trigger Summary

| Trigger | Purpose |
|----------|---------|
| pull_request | React to PR lifecycle |
| schedule | Run automatically on cron |
| workflow_dispatch | Manual execution |
| paths | Trigger only for selected files |
| paths-ignore | Ignore documentation/config changes |
| workflow_run | Chain workflows |
| repository_dispatch | Trigger from external systems |

---

# Key Learnings

- Understood the complete Pull Request lifecycle.
- Built validation workflows for PR quality checks.
- Automated recurring tasks using cron schedules.
- Optimized workflow execution with path filters.
- Learned how to chain workflows using `workflow_run`.
- Explored external event triggering with `repository_dispatch`.
- Gained deeper understanding of event-driven CI/CD pipelines in GitHub Actions.

---

# Repository

**GitHub Repository**

```
https://github.com/sopatel14/github-actions-practice
```

---

# Workflow Files

```
.github/workflows/pr-lifecycle.yml

.github/workflows/pr-checks.yml

.github/workflows/scheduled-tasks.yml

.github/workflows/smart-triggers.yml

.github/workflows/smart-triggers-ignore.yml

.github/workflows/tests.yml

.github/workflows/deploy-after-tests.yml

.github/workflows/external-trigger.yml
```

---

# Conclusion

Day 47 introduced advanced GitHub Actions triggers that make CI/CD pipelines more intelligent and event-driven. Instead of running workflows on every push, pipelines can now react to pull request activities, scheduled jobs, specific file changes, completion of other workflows, and even external applications. These capabilities are widely used in production environments to create scalable, automated, and efficient DevOps workflows.
