# Day 40 – My First GitHub Actions Workflow 🚀

## Objective
Today I created my first CI/CD pipeline using GitHub Actions and learned how automation runs in the cloud whenever I push code to GitHub.

https://github.com/sopatel14/github-actions-practice

---

## What I Built

I created a simple GitHub Actions workflow that:

- Triggers on every push
- Runs a job on an Ubuntu runner
- Prints messages and system information
- Demonstrates how CI/CD pipelines execute step by step

---

## Workflow File

Path: `.github/workflows/hello.yml`

```yaml
# Workflow [ No indentation required for naming the workflow ]
name: hello

# On tells when the workflow should run [ Give Indentation by Tab ] 

on: 
  push:

# Job tells what the workflows should do, there can be one ore more jobs, hence we write jobs
# Jobs is an object so it needs indentation

jobs:
 # 1st job [ Give indentation]
  say-hello:
 # Creates an ephemeral (that lasts only for short time) conatiner with ubuntu running  
    runs-on: ubuntu-latest
  # that ephermal container will have steps to execute, hence name those steps and use run: run has the command

    steps:

      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Say hello to everyone
        run: echo "Hello Dosto"
   
      - name: current date
        run: echo "Current date and time is $(date)"
    
      - name: Print the current branch
        run: echo running on branch ${GITHUB_REF##*/}
  
      - name: List files in the repository
        run: ls -al

      - name: Print OS name 
        run: echo "$RUNNER_OS"

      #  - name: Intentional Failure Step
      #   run: exit 1
```

<img width="3274" height="1168" alt="image" src="https://github.com/user-attachments/assets/ed608df3-77fd-48cb-b643-1fbb976b889c" />




GitHub Actions Concepts Explained

on:
Defines the event that triggers the workflow.
Here, it runs every time I push code.

jobs:
A workflow is made of one or more jobs.
Each job runs independently in a virtual environment.

runs-on:
Specifies the operating system for the runner.
Example: ubuntu-latest

steps:
A sequence of tasks executed inside the job.

uses:
Runs a predefined GitHub Action (like checking out code).

run:
Executes shell commands directly on the runner machine.

name:
Gives a readable label to each step for better logs.


### What I Learned:

How CI/CD pipelines are triggered automatically
How GitHub provides cloud runners for execution
How each step runs in sequence
How to use GitHub context variables like:
${{ github.ref_name }} → shows branch name
Failure Test (Optional Experiment)

I added a test failure step:

```
- name: Intentional Failure Step
  run: exit 1
```

### Result:

<img width="3340" height="1442" alt="image" src="https://github.com/user-attachments/assets/8f92f57a-1035-48f1-9b3b-9a45f4bc0066" />


The workflow turned ❌ red
Execution stopped at the failed step
Remaining steps did not run

How to debug:

Open GitHub → Actions tab
Click failed workflow
Expand failed step logs
Read error output carefully

### Conclusion

This was my first hands-on experience with CI/CD pipelines.

### Now I understand how:

Code push → triggers workflow
Workflow → runs automated steps
Logs → help debug failures
