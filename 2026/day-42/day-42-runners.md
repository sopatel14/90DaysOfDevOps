# Day 42 – GitHub Runners: GitHub-Hosted & Self-Hosted

https://github.com/sopatel14/github-actions-practice/tree/main

## Objective

The goal of Day 42 was to understand how GitHub Actions executes workflows using **runners**. I explored both **GitHub-hosted runners** provided by GitHub and **self-hosted runners** that run on my own machine. I also learned how to target specific runners using labels and compared the advantages and disadvantages of each approach.

---

# Task 1 – GitHub-Hosted Runners

## Workflow

**File:** `.github/workflows/git-hosted-runner.yml`


```yaml
name: GitHub Hoster Runners

on:
  push:

jobs:
  ubuntu-os:
    runs-on: ubuntu-latest

    steps:
      - name: Print the ubuntu-os info
        run: |
          uname -o
          hostname
          whoami

  windows-os:
    runs-on: windows-latest

    steps:
      - name: Print the windows-os info
        run: |
          cmd /c ver 
          hostname
          whoami

  macos-os:
    runs-on: macos-latest

    steps:
      - name: Print the macos-os info
        run: |
          uname -o 
          hostname
          whoami

```

## Output

<img width="3338" height="1310" alt="image" src="https://github.com/user-attachments/assets/d23462e6-d290-497c-a6e6-19271116d423" />

---

## What is a GitHub-Hosted Runner?

A **GitHub-hosted runner** is a virtual machine provided and managed by GitHub. Whenever a workflow is triggered, GitHub automatically creates a fresh virtual machine, executes the workflow, and destroys the machine after the job completes.

### Key Points

* Managed entirely by GitHub
* Automatically provisioned for every workflow run
* Comes with many development tools pre-installed
* No maintenance required
* Suitable for most CI/CD workloads

---

# Task 2 – Explore Pre-installed Software

## Workflow Step

`.github/workflows/pre-installed-software.yml`

```yaml
name: Pre-Installed Softwares

on:
  push:

jobs:
  softwares-on-ubuntu:
    runs-on: ubuntu-latest

    steps:
      - name: Check Docker version  
        run: docker --version

  
      - name: Check Python version  
        run: python3 --version
      
    
      - name: Check Node version  
        run: node -v

    
      - name: Check Git version  
        run: git --version

```

## Output

<img width="2548" height="1442" alt="image" src="https://github.com/user-attachments/assets/6a5bf3ce-f98e-4d3c-9fbb-ce9ffa82f75d" />


The following versions were verified on the Ubuntu runner:

* Docker Version:
* Python Version:
* Node.js Version:
* Git Version:

---

## Why do pre-installed tools matter?

GitHub-hosted runners already include many commonly used development tools such as Docker, Python, Node.js, Java, .NET, Git, and many others.

This is useful because:

* No time is spent installing basic software.
* Workflows execute faster.
* Configuration is simpler.
* Every workflow starts with a clean and consistent environment.
* Teams get reproducible builds across different executions.

---

# Task 3 – Setting Up a Self-Hosted Runner

I created a self-hosted runner from:

**Repository → Settings → Actions → Runners → New Self-hosted Runner**

I selected:

* Operating System: Linux
* Architecture: (your choice)

Then I downloaded, configured, and started the runner on my own machine.

---

## Screenshot

<img width="2712" height="1064" alt="image" src="https://github.com/user-attachments/assets/36003cec-e25c-4948-8866-86f2decab082" />


**Self-hosted Runner showing "Idle" (Green Status)**

<img width="3268" height="774" alt="image" src="https://github.com/user-attachments/assets/ee321ae6-f512-4c12-88f6-5badde33af6c" />


---

# Task 4 – Run Workflow on Self-Hosted Runner

## Workflow

**File:** `.github/workflows/self-hosted.yml`



```yaml
name: self-hosted runner

on:
  push:

jobs:
  self-hosted:
    runs-on: [self-hosted, my-linux-runner]

    steps:
      - name: Print the hostname  
        run: hostname

      - name: Print the working directory
        run: pwd

      - name: Create a file
        run: touch file-created.txt
```

The workflow performed the following actions:

* Printed the hostname
* Printed the working directory
* Created a file
* Verified the file exists

---

## Output

<img width="2424" height="1278" alt="image" src="https://github.com/user-attachments/assets/d9ab6d12-25ff-49a0-ac39-b7ce6094a94e" />


---


**Workflow running successfully on the Self-Hosted Runner**

<img width="1640" height="820" alt="image" src="https://github.com/user-attachments/assets/b7be0876-c815-48f1-b40a-96d5ec60bd9c" />


---

## Verification

The created file was successfully found on my local machine after the workflow completed, confirming that the workflow actually executed on my own hardware instead of GitHub's infrastructure.

<img width="729" height="66" alt="image" src="https://github.com/user-attachments/assets/7a7ef9e1-a87e-408a-8443-f94ca508abe7" />


---

# Task 5 – Runner Labels

I added the following custom label to my self-hosted runner:

```
my-linux-runner
```

Then I updated my workflow to use:

```yaml
jobs:
  self-hosted:
    runs-on: [self-hosted, my-linux-runner]
```

The workflow successfully executed on the labeled runner.

<img width="1132" height="494" alt="image" src="https://github.com/user-attachments/assets/6f87d889-996f-4720-8eff-50f6955dce09" />


---

## Why are labels useful?

Labels help GitHub decide which runner should execute a workflow.

They become especially useful when multiple self-hosted runners exist because they allow workflows to target specific machines based on:

* Operating system
* Hardware configuration
* Installed software
* Environment
* Team ownership
* Geographic location

Labels make runner selection more organized and scalable.

---

# GitHub-Hosted vs Self-Hosted Runners

| Feature                 | GitHub-Hosted Runner                                         | Self-Hosted Runner                                                                          |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------- |
| **Who manages it?**     | GitHub                                                       | User / Organization                                                                         |
| **Cost**                | Included with GitHub minutes (Free & Paid plans have limits) | User pays for hardware, VM, cloud instance, electricity, maintenance, etc.                  |
| **Pre-installed tools** | Large collection of tools already installed                  | User installs and maintains required software                                               |
| **Good for**            | General CI/CD, testing, building, open-source projects       | Custom environments, private infrastructure, large builds, GPU workloads, internal networks |
| **Security concern**    | GitHub manages security and isolation                        | User is responsible for OS updates, security patches, network security, and access control  |

---

# Key Learnings

* Every GitHub Actions job requires a runner to execute.
* GitHub-hosted runners are temporary virtual machines managed by GitHub.
* Self-hosted runners execute workflows on your own hardware or cloud servers.
* Self-hosted runners provide greater flexibility but require maintenance.
* Labels allow workflows to target specific runners in environments with multiple machines.
* Pre-installed software on GitHub-hosted runners reduces setup time and simplifies workflow configuration.

---

# Conclusion

Day 42 provided practical experience with the infrastructure behind GitHub Actions. I learned the difference between GitHub-hosted and self-hosted runners, configured my own self-hosted runner, executed workflows on my local machine, and used runner labels to target specific execution environments. This knowledge is essential for building scalable and production-ready CI/CD pipelines.
