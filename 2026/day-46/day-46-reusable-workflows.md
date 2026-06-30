# Day 46 – Reusable Workflows & Composite Actions

# Repository

**GitHub Repository:**

https://github.com/sopatel14/github-actions-practice

## Objective

Today's goal was to learn how to reduce duplication in GitHub Actions by creating reusable workflows and custom composite actions.

---

# Task 1 – Understanding `workflow_call`

## What is a Reusable Workflow?

A reusable workflow is a GitHub Actions workflow that can be called from another workflow. It helps eliminate duplicate CI/CD logic across multiple repositories or workflows.

---

## What is the `workflow_call` Trigger?

`workflow_call` is a special trigger that allows one workflow to invoke another workflow.

Unlike `push` or `pull_request`, a reusable workflow only runs when another workflow explicitly calls it.

---

## Reusable Workflow vs Regular Action

| Reusable Workflow | Regular Action |
|-------------------|----------------|
| Contains one or more jobs | Contains one or more steps |
| Triggered using `workflow_call` | Used inside a workflow step with `uses:` |
| Can use runners (`runs-on`) | Cannot define runners |
| Can accept inputs, secrets and return outputs | Can accept inputs and outputs |

---

## Where Must a Reusable Workflow Live?

```
.github/workflows/
```

---

# Task 2 – Creating a Reusable Workflow - .github/workflows/reusable-build.yml

Created:

```
name: Reusable Build Workflow

on:
  workflow_call:
    inputs:
      app_name:
        description: "Name of the application"
        required: true
        type: string

      environment:
        description: "Deployment environment"
        required: true
        default: staging
        type: string

    secrets:
      docker_token:
        required: true

    outputs:
      build_version:
        description: "Generated build version"
        value: ${{ jobs.build.outputs.build_version }}

jobs:
  build:
    runs-on: ubuntu-latest

    outputs:
      build_version: ${{ steps.generate_tag.outputs.VERSION }}

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Print Build Information
        run: |
          echo "Building ${{ inputs.app_name }} for ${{ inputs.environment }}"

      - name: Verify Docker Token
        run: |
          if [ -n "${{ secrets.docker_token }}" ]; then
            echo "Docker token is set: true"
          else
            echo "Docker token is set: false"
          fi
      - name: Generate Short SHA and Version String
        id: generate_tag
        run: |
          SHORT_SHA=$(echo "${{ github.sha }}" | cut -c1-7)
          VERSION_STRING="v1.0-${SHORT_SHA}"

          echo "Generated version: $VERSION_STRING"

          echo "VERSION=$VERSION_STRING" >> "$GITHUB_OUTPUT"
        
```

Features:

- Triggered using `workflow_call`
- Accepts inputs:
  - app_name
  - environment
- Accepts secret:
  - docker_token
- Checks out the repository
- Prints build information
- Verifies Docker secret without exposing it

---

# Task 3 – Creating a Caller Workflow - .github/workflows/call-build.yml

Created:

```
name: Workflow workflow_call

on:
  push:

jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
    with:
      app_name: "my-web-app"
      environment: "production"
    secrets:
      docker_token: ${{ secrets.DOCKER_TOKEN }}

  print-version:
    needs: build
    runs-on: ubuntu-latest

    steps:
      - name: Print Build Version
        run: |
          echo "Build Version: ${{ needs.build.outputs.build_version }}"

```

The workflow:

- Triggers on push
- Calls the reusable workflow
- Passes inputs
- Passes GitHub Secrets

Example:

```yaml
jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
```

---

# Task 4 – Passing Outputs

Generated a build version using the short Git commit SHA.

Example:

```
v1.0-a1b2c3d
```

Output Flow

```
Step Output
      │
      ▼
Job Output
      │
      ▼
Workflow Output
      │
      ▼
Caller Workflow
```

The caller workflow then printed the generated version using:

```yaml
needs.build.outputs.build_version
```

---

# Task 5 – Composite Action - .github/actions/setup-and-greet/action.yml


Created a custom composite action:

```

name: Setup and Greet
description: A custom composite action that greets the user

inputs:
  name:
    description: Name of the person
    required: true

  language:
    description: Language for greeting
    required: false
    default: en

outputs:
  greeted:
    description: Indicates the greeting was completed
    value: ${{ steps.greet.outputs.greeted }}

runs:
  using: composite

  steps:
    - name: Print Greeting
      id: greet
      shell: bash
      run: |
        if [ "${{ inputs.language }}" = "en" ]; then
          echo "Hello, ${{ inputs.name }}!"
        elif [ "${{ inputs.language }}" = "es" ]; then
          echo "Hola, ${{ inputs.name }}!"
        elif [ "${{ inputs.language }}" = "fr" ]; then
          echo "Bonjour, ${{ inputs.name }}!"
        else
          echo "Hello, ${{ inputs.name }}!"
        fi

        echo "greeted=true" >> "$GITHUB_OUTPUT"

    - name: Print Date and Runner OS
      shell: bash
      run: |
        echo "Current Date: $(date)"
        echo "Runner OS: $RUNNER_OS"
```

The action:

- Accepts:
  - name
  - language
- Prints greeting
- Prints current date
- Prints runner operating system
- Returns output:
  - greeted = true

### Created another workflow to execute the composite action. - .github/workflows/composite-action.yml


```yaml

name: Composite Action Demo

on:
  workflow_dispatch:

jobs:
  greet:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Run Custom Composite Action
        id: greeting
        uses: ./.github/actions/setup-and-greet
        with:
          name: Sourav
          language: en

      - name: Print Action Output
        run: |
          echo "Greeting completed: ${{ steps.greeting.outputs.greeted }}"
```

---

# Task 6 – Reusable Workflow vs Composite Action

| Feature | Reusable Workflow | Composite Action |
|----------|-------------------|------------------|
| Triggered by | `workflow_call` | `uses:` inside a workflow step |
| Can contain jobs? | ✅ Yes | ❌ No |
| Can contain multiple steps? | ✅ Yes | ✅ Yes |
| Lives where? | `.github/workflows/` | `.github/actions/<action-name>/action.yml` |
| Can accept secrets directly? | ✅ Yes | ❌ No (passed as inputs or environment variables) |
| Best for | Complete CI/CD pipelines | Reusable groups of steps |

---

# Files Created

```
.github/workflows/reusable-build.yml
.github/workflows/call-build.yml
.github/workflows/composite-action.yml

.github/actions/setup-and-greet/action.yml
```

---

# Folder Structure

```
.github
├── actions
│   └── setup-and-greet
│       └── action.yml
│
└── workflows
    ├── reusable-build.yml
    ├── call-build.yml
    └── composite-action.yml
```

---

# Key Learnings

- Learned how reusable workflows eliminate duplicate pipeline logic.
- Understood how `workflow_call` works.
- Passed inputs and secrets between workflows.
- Generated and consumed workflow outputs.
- Created a custom composite action.
- Learned the differences between reusable workflows and composite actions.

---

# Screenshots

## Reusable Workflow

<img width="3352" height="1668" alt="image" src="https://github.com/user-attachments/assets/b2c7c6b4-aa68-4e21-ac07-00fcb4fd9ef1" />

---

## Caller Workflow

<img width="3350" height="1610" alt="image" src="https://github.com/user-attachments/assets/49b8b43e-b514-4a1d-9ae7-b2bcafbea342" />

---

## Workflow Outputs

<img width="3360" height="1620" alt="image" src="https://github.com/user-attachments/assets/0000e3df-3c9e-42ea-ac78-3f3bfda56168" />

---

## Composite Action Execution

<img width="3360" height="1678" alt="image" src="https://github.com/user-attachments/assets/35edd465-7e68-4a35-9b32-ecffefbf905f" />
