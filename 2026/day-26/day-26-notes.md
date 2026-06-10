# Day 26 – GitHub CLI (gh)

## Overview

GitHub CLI (`gh`) allows developers and DevOps engineers to manage repositories, issues, pull requests, releases, and workflows directly from the terminal without switching to the browser.

---

# Task 1: Install and Authenticate

## Installation

### Ubuntu

```bash
sudo apt update
sudo apt install gh -y
```

## Authenticate

```bash
gh auth login
```

Follow the prompts to authenticate using GitHub.

## Verify Login

```bash
gh auth status
```

### Observation

Shows the currently authenticated GitHub account and authentication status.

### Question: What authentication methods does gh support?

GitHub CLI supports:

* Browser-based authentication
* Personal Access Token (PAT)
* SSH authentication
* GitHub Enterprise authentication

---

# Task 2: Working with Repositories

## Create Repository

```bash
gh repo create gh-test-repo --public --clone --add-readme
```
### Output:

<img width="1130" height="246" alt="image" src="https://github.com/user-attachments/assets/d82b1b6c-f089-40be-951e-40c6cd94aaed" />


## Clone Repository

```bash
gh repo clone owner/repository-name
```

### Output

<img width="1140" height="326" alt="image" src="https://github.com/user-attachments/assets/e426b4fc-d15f-4d07-a3b3-3b72a1124281" />


## View Repository Details

```bash
gh repo view
```

## List All Repositories

```bash
gh repo list
```

### Output

<img width="1140" height="538" alt="image" src="https://github.com/user-attachments/assets/d9c7af16-dc18-4a2a-8f6c-64383dc5c8f4" />


## Open Repository in Browser

```bash
gh repo view --web
```

## Delete Repository

```bash
gh repo delete gh-test-repo
```
### Output

<img width="1118" height="142" alt="image" src="https://github.com/user-attachments/assets/5d589ad2-52ad-4176-b654-28eb7035dba1" />


### Observation

Repository creation, cloning, viewing, and deletion can be completed entirely from the terminal.

---

# Task 3: Issues

## Create Issue

```bash
gh issue create \
  --title "Test Issue" \
  --body "Created using GitHub CLI" \
  --label bug
```

## List Open Issues

```bash
gh issue list
```

## View Specific Issue

```bash
gh issue view 1
```

## Close Issue

```bash
gh issue close 1
```

### Output

<img width="1360" height="684" alt="image" src="https://github.com/user-attachments/assets/211a4377-b26c-4037-a157-cbfece86a31a" />


### Question: How could you use gh issue in automation?

Possible use cases:

* Automatically create issues when monitoring scripts detect failures.
* Generate incident tickets from CI/CD pipelines.
* Create recurring maintenance tasks.
* Report infrastructure problems from automation scripts.

---

# Task 4: Pull Requests

## Create Feature Branch

```bash
git checkout -b feature-gh-cli
```

## Make Changes

```bash
echo "GitHub CLI Practice" >> notes.txt
git add .
git commit -m "Added notes"
git push -u origin feature-gh-cli
```

## Create Pull Request

```bash
gh pr create --fill
```

## List Pull Requests

```bash
gh pr list
```

## View Pull Request

```bash
gh pr view
```

## Merge Pull Request

```bash
gh pr merge
```

### Output


<img width="1366" height="520" alt="image" src="https://github.com/user-attachments/assets/dd5e4dbc-d428-496f-8cae-3794d764b6a2" />


### Question: What merge methods does gh pr merge support?

```bash
gh pr merge --merge
gh pr merge --squash
gh pr merge --rebase
```

Supported methods:

* Merge Commit
* Squash Merge
* Rebase Merge

### Question: How would you review someone else's PR using gh?

```bash
gh pr checkout <pr-number>
gh pr diff <pr-number>
gh pr review <pr-number> --approve
gh pr review <pr-number> --comment
gh pr review <pr-number> --request-changes
```

---

# Task 5: GitHub Actions & Workflows

## List Workflow Runs

```bash
gh run list
```

## View Workflow Run Details

```bash
gh run view <run-id>
```

### Question: How could gh run and gh workflow help in CI/CD?

Benefits:

* Monitor pipeline status from terminal.
* Trigger workflows remotely.
* Debug failed runs.
* Automate deployment checks.
* Integrate workflow monitoring into scripts.

---

# Task 6: Useful gh Commands

## GitHub API

```bash
gh api user
```

## Gists

```bash
gh gist create notes.txt
gh gist list
```

## Releases

```bash
gh release create v1.0.0
```

## Aliases

```bash
gh alias set prs "pr list"
gh prs
```

## Search Repositories

```bash
gh search repos devops
```

---

# Key Takeaways

* GitHub CLI reduces context switching.
* Most GitHub operations can be completed from the terminal.
* JSON output enables automation and scripting.
* Useful for DevOps workflows, CI/CD monitoring, and repository management.
* Pull requests, issues, workflows, and releases can all be managed without using the browser.
