# Day 43: Controlling Pipeline Flow — Jobs, Steps, Env Vars & Conditionals

## 🎯 Task Overview
Today's objective was to master the flow control mechanics of GitHub Actions workflows. This included structuring multi-job dependency graphs, managing cascading environment variable scopes, parsing runtime contexts, passing sequential data between isolated runners via step outputs, and defining dynamic runtime execution filters using status checks and branches.

---

## 🚀 Challenge Tasks & Workflow Snippets

### Task 1: Multi-Job Workflow (`.github/workflows/multi-job.yml`)
This workflow demonstrates how to run independent jobs sequentially instead of in parallel using the `needs` keyword configuration gate.

```yaml
name: Multi Jobs 

on: 
  push:


jobs:
  code-checkout-build:

    runs-on: ubuntu-latest

    steps:

      - name: Code checkout
        uses: actions/checkout@v4

      - name: Code Building
        run: echo "Building the app"

  code-test:
    runs-on: ubuntu-latest
    needs: [code-checkout-build]
    steps:

      - name: Code Testing 
        run: echo "Running tests"

  code-deploy:
    runs-on: ubuntu-latest
    needs: [code-test] 
    steps:

      - name: Code Deploying
        run: echo "Deploying"
```

#### 📸 Task 1 Execution Verification Graph
<img width="3358" height="1256" alt="image" src="https://github.com/user-attachments/assets/55183d01-69e9-4599-824c-46b9e79f3ef1" />


---

### Task 2: Environment Variables (`.github/workflows/env-variables.yml`)
This challenge maps the three structural hierarchical scopes of environment variables (Global/Workflow, Job-Specific, and Local/Step-Specific) alongside system context parameters.

```yaml
name: priting env variables

on: 
  push:

env:
  APP_NAME: myapp

jobs:
  print-env-variables:
    runs-on: ubuntu-latest


    env:
      ENVIRONMENT: staging

    steps:
      - name: Print everything

        env:
          VERSION: 1.0.0

        run: |
          echo "=== Environment Variables ==="
          echo "App Name: $APP_NAME"
          echo "Environment: $ENVIRONMENT"
          echo "Version: $VERSION"
          
          echo "=== GitHub Context Variables ==="
          echo "Commit SHA: ${{ github.sha }}"
          echo "Actor: ${{ github.actor }}"
```

#### 📸 Task 2 Runtime Log Output
<img width="3360" height="1392" alt="image" src="https://github.com/user-attachments/assets/a13ae2c7-20f0-45e4-afcc-2fee59818919" />

---

### Task 3: Job Outputs (`.github/workflows/job-outputs.yml`)
An implementation showing how data generated dynamically inside one runner machine can be exposed to a completely separate job environment down the line.

```yaml
name: Job Outputs

on:
  push:

jobs:
  build-date:
    runs-on: ubuntu-latest
    outputs:
      date_output: ${{ steps.date_step.outputs.computed_date }}

    steps:
      - name: Generate Today's Date
        id: date_step
        # This bash command generates a date string and appends it to the special GitHub Environment File
        run: echo "computed_date=$(date +'%Y-%m-%d')" >> $GITHUB_OUTPUT

  print-date:
    runs-on: ubuntu-latest
    needs: [build-date]
    
    steps:
      - name: Print Received Date Output
        run: echo "The date passed from the previous job is ${{ needs.build-date.outputs.date_output }}"
```

#### 📸 Task 3 Data Inter-Job Passing Output

<img width="3360" height="1460" alt="image" src="https://github.com/user-attachments/assets/bc160bd7-bfb1-4e6c-b8fa-5767766ced68" />

<img width="3360" height="1600" alt="image" src="https://github.com/user-attachments/assets/adf357f7-0906-4cf3-a601-013e5f7d2fe5" />

---

### Task 4: Conditionals (`.github/workflows/conditionals-events.yml`)
This workflow implements custom status flags, branch constraints, and soft failure ignores to modify pipeline flows conditionally.

```yaml
name: Conditional events

on: 
  push:

jobs:
  conditional-event:
    runs-on: ubuntu-latest

    steps:
      - name: Only run If branch is main
        if: github.ref == 'refs/heads/main'
        run: echo "Successfully running on main"

      - name: Force an intentional error
        run: exit 1

      - name: Run only if previous step fails
        if: failure()
        run: echo "Previous Job failed"

      - name: Continue running on errors 
        continue-on-error: true
        run: echo "Code deployed without exiting"
```

#### 📸 Task 4 Branch & Status Filter Logs

<img width="3350" height="1574" alt="image" src="https://github.com/user-attachments/assets/40e292e2-d0b9-4b4a-bb29-1d53b078266a" />

---

### Task 5: Putting It Together (`.github/workflows/smart-pipeline.yml`)
The final blueprint sets up parallel execution blocks that eventually converge on a single reporting job that dynamically identifies the pushing branch and outputs the commit metadata.

```yaml
name: Parallel and Summary Workflow

on:
  push:

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - name: Code Linting
        run: echo "Running linter checks..."

  test:
    runs-on: ubuntu-latest
    steps:
      - name: Code Testing
        run: echo "Running suite of tests..."

  summary:
    runs-on: ubuntu-latest
    needs: [lint, test]
    steps:
      - name: Print Run Details
        run: |
          echo "=== Branch Classification ==="
          echo "${{ github.ref == 'refs/heads/main' && 'This is a main branch push!' || 'This is a feature branch push!' }}"
          
          echo "=== Commit Information ==="
          echo "Commit Message: ${{ github.event.head_commit.message }}"
```

#### 📸 Task 5 Execution Visual Flow & Summary Logs

<img width="3360" height="1630" alt="image" src="https://github.com/user-attachments/assets/45bc2344-e3aa-419e-9e17-f6e80c0b3994" />

<img width="1678" height="706" alt="image" src="https://github.com/user-attachments/assets/842c57bb-2fa4-4a14-8edd-1b5f522ccd2e" />

---

## 🧠 Architectural Insights (Theory Section)

### What does `needs:` do?
`needs:` makes a job wait for another job to finish successfully before it starts.

### What does `outputs:` do?
`outputs:` allow one job to share data with another job running on a different runner.

### Why pass outputs between jobs?
Jobs are isolated, so outputs are used to transfer runtime values like version numbers or IDs.

### What does `continue-on-error: true` do?
It lets a step fail without stopping the rest of the job, allowing the workflow to continue.
