# Day 41 — GitHub Actions: Triggers & Matrix Builds

> **Repo:** `https://github.com/sopatel14/github-actions-practice` 

---

## Task 1: Pull Request Trigger

### `.github/workflows/pr-check.yml`

```yaml

# Goal Trigger it only when a pull request is opened or updated against main

name: Trigger-Pull-request

on:
  pull_request: 
    branches: 
      - main

jobs:
  validate-pr:
    runs-on: ubuntu-latest


    steps:
      - name: checkout repository
        uses: actions/checkout@v4
          
      - name: Print the Branch name
        run: echo "PR check running for branch:${{ github.head_ref }}"

```

### Screenshot — Workflow running on PR page

<img width="2110" height="456" alt="image" src="https://github.com/user-attachments/assets/907b510a-9e89-4d5c-95fc-c92d4e917959" />

<img width="3334" height="1452" alt="image" src="https://github.com/user-attachments/assets/cedc4a28-0e88-4f73-a722-d3ed50ab4f93" />


---

## Task 2: Scheduled Trigger

### Schedule block added to workflow

```yaml
# What is the goal for your workflow
# Goal: Whenever someones change the python code
# The liner will check if the syntax, format, indentation(linting) is correct or not

name: Auto linter

on:
  push:
    paths: "*.py"

# schedule:
#     - cron: "0 0 * * *"  # every day at midnight UTC

jobs:
  code-linter:
    runs-on: ubuntu-latest
    # this step will automatically clone the code of the current branch into the runner
    steps:
      - name: Checkout/Clone the code
        uses: actions/checkout@v

    # this step will install python
      - name: Setup python
        uses: actions/setup-python@v5
        with:
          python-version: '3.14'
        
    # this step will install the linting tool
      - name: Install Linter
        run: pip install ruff

    # this step will run the linter
      - name: Check the files for linting
        run: ruff check .

```

### Cron Expression Answer

**Every day at midnight UTC:**
```
0 0 * * *
```

**Every Monday at 9 AM UTC:**
```
0 9 * * 1
```

> **Breakdown:** `0` (minute 0) `9` (hour 9) `*` (any day of month) `*` (any month) `1` (Monday)

---

## Task 3: Manual Trigger (`workflow_dispatch`)

### `.github/workflows/manual.yml`

```yaml
# # Goal : Create .github/workflows/manual.yml with a workflow_dispatch: trigger
# Add an input that asks for an environment name (staging/production)
# Print the input value in a step


name: Trigger

on: 
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        required: true
        description: Environment
        options:
          - ''
          - development
          - staging 
          - production
        default: development

jobs:
  my_job:
    runs-on: ubuntu-latest

    steps:
      - name: Print selected environment
        run: echo "Selected environment:${{ inputs.environment }}"
```

### Screenshot — Manual trigger in Actions tab

<img width="3314" height="1260" alt="image" src="https://github.com/user-attachments/assets/2205d4d9-76a0-49a2-84ab-62b56d781f89" />

<img width="3344" height="1368" alt="image" src="https://github.com/user-attachments/assets/46f2226d-9754-44b2-bab0-4d34f5575f72" />

---

## Task 4: Matrix Builds

### `.github/workflows/matrix.yml` — Python versions only (3 jobs)

```yaml
# # Goal Uses a matrix strategy to run the same job across:
# Python versions: 3.10, 3.11, 3.12
# Each job installs Python and prints the version
# Watch all 3 run in parallel

name: Matrix-strategy

on:
  push:

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:

      fail-fast: false

      matrix:
        os: [ubuntu-latest, windows-latest]
        python-version: ["3.10", "3.11", "3.12"]

        exclude:
          - os: windows-latest
            python-version: "3.10"  

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: Print Python version
        run: python --version

```

### Screenshot — 3 parallel jobs running

<img width="1868" height="826" alt="image" src="https://github.com/user-attachments/assets/d1de0413-b018-49a1-a802-68b8e5c2c3d0" />

<img width="3360" height="1300" alt="image" src="https://github.com/user-attachments/assets/5c77f168-8fde-431d-b644-338959aa8e5f" />


---


### Screenshot — 6 parallel jobs running

<img width="3350" height="1178" alt="image" src="https://github.com/user-attachments/assets/bf48c43f-0d8f-4b3d-8df6-4f1c7439761a" />

---

## Task 5: Exclude & Fail-Fast


### Screenshot — One job failing, others continuing

<img width="3344" height="1484" alt="image" src="https://github.com/user-attachments/assets/393b29b5-ceb8-47f8-b4c8-92e5a412a84f" />


---

### Notes: `fail-fast: true` vs `fail-fast: false`

| Setting | Behaviour |
|---|---|
| `fail-fast: true` _(default)_ | As soon as **any** job in the matrix fails, GitHub cancels all remaining in-progress jobs immediately. Saves CI minutes but you lose results for the other combinations. |
| `fail-fast: false` | Every job in the matrix **runs to completion** regardless of other failures. You get full results across all combinations — useful for knowing exactly which envs are broken. |

---

## Summary

| Task | Trigger type | Key syntax |
|---|---|---|
| 1 | Pull request | `on: pull_request: branches: [main]` |
| 2 | Schedule (cron) | `on: schedule: - cron: '0 0 * * *'` |
| 3 | Manual dispatch | `on: workflow_dispatch: inputs:` |
| 4 | Matrix build | `strategy: matrix: python-version: [...]` |
| 5 | Exclude + fail-fast | `exclude:` + `fail-fast: false` |
