# Day 38 – YAML Basics

## Objective

Today's goal was to understand YAML syntax and structure, which is the foundation of modern CI/CD pipelines such as GitHub Actions, Jenkins, GitLab CI/CD, ArgoCD, Kubernetes manifests, and Ansible playbooks.

By the end of this exercise, I learned how to create YAML files, define lists and nested objects, use multi-line strings, and validate YAML syntax.

---

# Task 1: Key-Value Pairs

Created a file named `person.yaml`.

### person.yaml

```yaml
name: Sourav
role: DevOps Engineer
experience_years: 2
learning: true
```

<img width="458" height="128" alt="image" src="https://github.com/user-attachments/assets/bce33024-3d0f-4bda-b185-9e64fa8728ee" />


### Notes

* YAML uses `key: value` pairs.
* Booleans are written as `true` or `false`.
* YAML is sensitive to indentation.
* Tabs should never be used.

---

# Task 2: Lists

Updated `person.yaml` to include tools and hobbies.

### Updated person.yaml

```yaml
name: Sourav Patel
role: DevOps Engineer
experience_years: 1
learning: true

tools:
  - Linux
  - Docker
  - K8s
  - Github  
  - Terraform

hobbies: [Learning DevOps, Reading, Cricket]
```

<img width="472" height="309" alt="image" src="https://github.com/user-attachments/assets/ab9b41eb-2f13-48ed-acb4-7b7803f24918" />


### Two Ways to Write Lists in YAML

### Block Style

```yaml
tools:
  - Docker
  - Git
  - Jenkins
```

### Inline Style

```yaml
tools: [Docker, Git, Jenkins]
```

### Notes

Block style is more readable for long lists.

Inline style is useful for short lists.

---

# Task 3: Nested Objects

Created a file named `server.yaml`.

### server.yaml

```yaml
server:
  name: web-server
  ip: 192.168.1.10
  port: 8080

database:
  host: localhost
  name: appdb

  credentials:
    user: admin
    password: secret123
```

<img width="491" height="275" alt="image" src="https://github.com/user-attachments/assets/df1f304f-4739-4fa6-b1e7-f98909e06374" />


### Notes

Nested objects are created using indentation.

Each level is typically indented using 2 spaces.

Example:

```yaml
parent:
  child:
    grandchild: value
```

### Experiment

When a tab character was used instead of spaces, YAML validation failed because YAML only supports spaces for indentation.

---

# Task 4: Multi-line Strings

Added startup scripts using both YAML block styles.

### server.yaml

```yaml
server:
  name: web-server
  ip: 192.168.1.10
  port: 8080

database:
  host: localhost
  name: appdb

  credentials:
    user: admin
    password: secret123

startup_script_preserve: |
  #!/bin/bash
  echo "Starting application"
  docker compose up -d
  echo "Application started"

startup_script_folded: >
  This startup script is used
  to initialize the application
  during server boot.
```

<img width="518" height="416" alt="image" src="https://github.com/user-attachments/assets/375ee701-3354-4179-953d-220090bd9616" />


### Difference Between `|` and `>`

#### Pipe (`|`)

Preserves line breaks exactly as written.

```yaml
message: |
  Line 1
  Line 2
```

Output:

```text
Line 1
Line 2
```

#### Greater Than (`>`)

Converts line breaks into spaces.

```yaml
message: >
  Line 1
  Line 2
```

Output:

```text
Line 1 Line 2
```

### When to Use Them

| Symbol | Use Case                                        |                                                                  |
| ------ | ----------------------------------------------- | ---------------------------------------------------------------- |
| `      | `                                               | Shell scripts, configuration files, certificates, multiline logs |
| `>`    | Long descriptions, documentation text, comments |                                                                  |

---

# Task 5: Validate YAML

### Validation Tool

Used:

```bash
yamllint person.yaml
yamllint server.yaml
```

or

https://yamllint.com

### Example Error

Broken YAML: Extra space in the file

<img width="540" height="137" alt="image" src="https://github.com/user-attachments/assets/6366f761-af86-4ed0-b030-fca1fd5066c6" />



### Fix

Replace tabs with spaces:

```yaml
server:
  name: web-server
```

Validation passed successfully after correction.

---

# Task 6: Spot the Difference

### Correct YAML

```yaml
name: devops

tools:
  - docker
  - kubernetes
```

### Broken YAML

```yaml
name: devops

tools:
- docker
  - kubernetes
```

### What's Wrong?

The list items are not properly indented under the `tools` key.

YAML expects list items to align consistently beneath the parent key.

Correct version:

```yaml
tools:
  - docker
  - kubernetes
```

---

# Key Learnings

### 1. Indentation Controls Structure

YAML does not use braces or brackets like JSON.

Structure is determined entirely by spacing.

### 2. YAML Uses Spaces Only

Tabs are invalid and will cause validation failures.

Using 2 spaces per indentation level is the standard practice.

### 3. Lists and Nested Objects Are Common

Most DevOps tools use YAML for:

* GitHub Actions workflows
* Kubernetes manifests
* Docker Compose files
* Ansible playbooks
* GitLab CI/CD pipelines

Understanding lists and nested objects is essential before learning CI/CD pipelines.

---

# Real-World DevOps Relevance

YAML is used extensively across the DevOps ecosystem:

* GitHub Actions (`.github/workflows/*.yml`)
* Docker Compose (`docker-compose.yml`)
* Kubernetes (`deployment.yaml`)
* Ansible (`playbook.yml`)
* ArgoCD Applications
* GitLab CI/CD (`.gitlab-ci.yml`)

Mastering YAML is a prerequisite for building reliable CI/CD pipelines and infrastructure-as-code solutions.

---

# Conclusion

Today I learned the fundamentals of YAML, including key-value pairs, lists, nested objects, multi-line strings, and validation. YAML is simple to read but highly sensitive to indentation, making validation an important part of the workflow. These concepts will be essential for upcoming CI/CD and Infrastructure as Code projects.
