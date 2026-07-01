Day 47 - Advanced Triggers: PR Events, Cron Schedules & Event-Driven Pipelines
Overview
This document covers advanced GitHub Actions triggers including PR lifecycle events, scheduled workflows, path filters, workflow chaining, and external event triggers.

Task 1: Pull Request Event Types
Workflow: pr-lifecycle.yml
yaml
name: PR Lifecycle Events

on:
  pull_request:
    types:
      - opened
      - synchronize
      - reopened
      - closed

jobs:
  pr-lifecycle:
    runs-on: ubuntu-latest
    
    steps:
      - name: Print PR event information
        run: |
          echo "========================================"
          echo "Pull Request Event Information"
          echo "========================================"
          echo "Event type: ${{ github.event.action }}"
          echo "PR Title: ${{ github.event.pull_request.title }}"
          echo "PR Author: ${{ github.event.pull_request.user.login }}"
          echo "Source branch: ${{ github.event.pull_request.head.ref }}"
          echo "Target branch: ${{ github.event.pull_request.base.ref }}"
          echo "========================================"
      
      - name: Check if PR was merged
        if: github.event.pull_request.merged == true
        run: |
          echo "========================================"
          echo "PR was merged successfully!"
          echo "PR #${{ github.event.pull_request.number }} merged"
          echo "========================================"
Task 2: PR Validation Workflow
Workflow: pr-checks.yml
yaml
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
Task 3: Scheduled Workflows
Workflow: scheduled-tasks.yml
yaml
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
Cron Expressions
Description	Cron Expression
Every weekday at 9 AM IST	30 3 * * 1-5
First day of every month at midnight	0 0 1 * *
Why scheduled workflows may be delayed or skipped:

Repository inactivity (no commits for 60+ days)

GitHub resource optimization for inactive repositories

Workload balancing across GitHub's runner infrastructure

Archived repositories do not run scheduled workflows

Task 4: Path & Branch Filters
Workflow: smart-triggers.yml
yaml
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
        with:
          fetch-depth: 0
      
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
          if [ "${{ github.event.before }}" != "0000000000000000000000000000000000000000" ]; then
            git diff --name-only ${{ github.event.before }} ${{ github.sha }}
          else
            git ls-files
          fi
          echo "========================================"
Workflow: smart-triggers-ignore.yml
yaml
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
        with:
          fetch-depth: 0
      
      - name: Show trigger information
        run: |
          echo "========================================"
          echo "Workflow triggered (paths-ignore filter)"
          echo "Branch: ${{ github.ref_name }}"
          echo "Commit: ${{ github.sha }}"
          echo "Commit message: ${{ github.event.head_commit.message }}"
          echo "========================================"
      
      - name: Show changed files
        run: |
          echo "Files changed in this push:"
          if [ "${{ github.event.before }}" != "0000000000000000000000000000000000000000" ]; then
            git diff --name-only ${{ github.event.before }} ${{ github.sha }}
          else
            git ls-files
          fi
          echo "========================================"
When to Use paths vs paths-ignore
Use Case	Use	Why
Only run tests when code changes	paths: ['src/**', 'app/**']	Don't waste time on documentation
Skip deployment when docs change	paths-ignore: ['*.md', 'docs/**']	Documentation doesn't need deployment
Run security scan on code files	paths: ['*.js', '*.py', '*.go']	Only scan relevant file types
Skip CI when only config changes	paths-ignore: ['*.yml', '*.yaml']	Configuration changes don't need tests
Run on specific folder changes	paths: ['backend/**']	Microservice-specific triggers
Note: You cannot use both paths and paths-ignore together in the same workflow. They must be in separate workflows.

Task 5: workflow_run - Chain Workflows Together
Workflow: tests.yml
yaml
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
Workflow: deploy-after-tests.yml
yaml
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
Explanation: workflow_run vs workflow_call
workflow_run:

Triggers automatically when another workflow completes

Runs on the default branch (main)

Can access the triggering workflow's conclusion (success/failure)

Use when you want to chain workflows based on completion status

Example: Deploy after tests pass, or notify on failure

workflow_call:

Triggers manually when called from another workflow

Allows passing inputs between workflows

Must be explicitly called using the uses keyword

Use when you want to reuse workflows and pass parameters

Example: Reusable deployment workflow called with different environments

Task 6: repository_dispatch - External Event Triggers
Workflow: external-trigger.yml
yaml
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
Triggering repository_dispatch
Using GitHub CLI (gh):

bash
gh api repos/sopatel14/github-actions-practice/dispatches \
  -f event_type=deploy-request \
  --raw-field client_payload='{"environment":"production"}'
Using curl with PAT:

bash
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: token ghp_YOUR_TOKEN" \
  https://api.github.com/repos/sopatel14/github-actions-practice/dispatches \
  -d '{"event_type":"deploy-request","client_payload":{"environment":"production"}}'
When to Use repository_dispatch
Use Case	Example
Slack bot triggers deployment	User types "/deploy production" in Slack
Monitoring tool triggers rollback	System detects error and triggers rollback
CI/CD pipeline from another system	Jenkins triggers GitHub Action after build
Scheduled external jobs	External cron triggers deployment
Webhook from another service	Third-party service triggers workflow
Summary Table of All Triggers
Trigger Type	File	Purpose
pull_request	pr-lifecycle.yml	PR lifecycle events
pull_request	pr-checks.yml	PR validation and gates
schedule	scheduled-tasks.yml	Cron-based automation
push with paths	smart-triggers.yml	Path-based triggers
push with paths-ignore	smart-triggers-ignore.yml	Path exclusion triggers
workflow_run	deploy-after-tests.yml	Workflow chaining
repository_dispatch	external-trigger.yml	External event triggers

Key Learnings
PR Events - Different lifecycle events allow granular control over when workflows run

Scheduled Workflows - Cron schedules enable automation without manual triggers

Path Filters - Optimize workflow execution by limiting triggers to relevant files

Workflow Chaining - workflow_run creates dependency between workflows

External Triggers - repository_dispatch enables integration with external systems

Notes
Scheduled workflows only run on the default branch (main)

paths-ignore takes precedence over paths when both are present

workflow_run gives access to the triggering workflow's conclusion and artifacts

repository_dispatch requires a personal access token with repo scope

Path filters use glob patterns - ** matches nested directories
