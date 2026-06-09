# Day 25 – Git Reset vs Revert & Branching Strategies

## Objective

Learn how to safely undo changes in Git using Reset and Revert, and understand common branching strategies used by software teams.

---

# Task 1: Git Reset

## Overview

Git Reset moves the current branch pointer to a previous commit. Depending on the option used, it may keep or remove staged and working directory changes.

---

## git reset --soft

### Command

```bash
git reset --soft HEAD~1
```

### Observation

* The last commit was removed.
* Changes remained staged.
* Files were ready to commit again.

### Output

<img width="609" height="1050" alt="image" src="https://github.com/user-attachments/assets/dc86f11c-e25f-499c-ba28-4e211b34f270" />


---

## git reset --mixed

### Command

```bash
git reset --mixed HEAD~1
```

### Observation

* The last commit was removed.
* Changes remained in the working directory.
* Files became unstaged.

### Output

<img width="725" height="631" alt="image" src="https://github.com/user-attachments/assets/8e688ffe-2431-4c60-b17e-c8e20018dea6" />


---

## git reset --hard

### Command

```bash
git reset --hard HEAD~1
```

### Observation

* The last commit was removed.
* All associated changes were deleted.
* Working directory matched the selected commit.

### Output

<img width="667" height="656" alt="image" src="https://github.com/user-attachments/assets/d61218d9-f446-49a9-b531-b21f1ca9cab9" />


---

## Answers

### What is the difference between --soft, --mixed, and --hard?

| Reset Type | Commit Removed | Changes Kept | Staged |
| ---------- | -------------- | ------------ | ------ |
| --soft     | Yes            | Yes          | Yes    |
| --mixed    | Yes            | Yes          | No     |
| --hard     | Yes            | No           | No     |

### Which one is destructive and why?

`git reset --hard` is destructive because it permanently removes commits and local changes.

### When would you use each one?

* **--soft**: Modify the previous commit.
* **--mixed**: Uncommit changes but keep files for editing.
* **--hard**: Completely discard commits and changes.

### Should you use reset on pushed commits?

Generally no. Reset rewrites history and can create problems for collaborators.

---

# Task 2: Git Revert

## Overview

Git Revert creates a new commit that undoes the changes from a previous commit.

### Command

```bash
git revert <commit-hash>
```

### Observation

* Git created a new commit.
* The original commit remained in history.
* The changes introduced by that commit were reversed.

### Output

<img width="645" height="584" alt="image" src="https://github.com/user-attachments/assets/7a873501-5169-4de7-a644-2ddc0834c9e5" />


---

## Answers

### How is git revert different from git reset?

* Reset moves branch history backward.
* Revert creates a new commit that undoes changes.

### Why is revert considered safer?

Because it preserves commit history and does not rewrite existing commits.

### When would you use revert vs reset?

* Use **revert** on shared branches.
* Use **reset** for local cleanup before pushing.

---

# Task 3: Reset vs Revert Comparison

| Feature                     | git reset                     | git revert                               |
| --------------------------- | ----------------------------- | ---------------------------------------- |
| What it does                | Moves branch pointer backward | Creates a new commit that undoes changes |
| Removes commit from history | Yes                           | No                                       |
| Rewrites history            | Yes                           | No                                       |
| Safe for shared branches    | No                            | Yes                                      |
| Safe for pushed commits     | No                            | Yes                                      |
| Typical use case            | Local cleanup                 | Undoing mistakes in shared repositories  |

---

# Task 4: Branching Strategies

## 1. GitFlow

### How It Works

Uses dedicated branches for development, features, releases, and hotfixes.

### Flow

```text
main
 ├── hotfix/*
 │
develop
 ├── feature/*
 └── release/*
```

### Used By

Teams with scheduled releases and structured workflows.

### Pros

* Highly organized
* Clear release process
* Good for large teams

### Cons

* More complex
* Slower delivery

---

## 2. GitHub Flow

### How It Works

Developers create short-lived feature branches from main and merge using pull requests.

### Flow

```text
main
 ├── feature-login
 ├── feature-api
 └── feature-dashboard
```

### Used By

Startups, SaaS products, and continuous deployment environments.

### Pros

* Simple
* Fast development
* Easy collaboration

### Cons

* Requires strong testing practices

---

## 3. Trunk-Based Development

### How It Works

Developers commit directly to main or use extremely short-lived branches.

### Flow

```text
main
 ├── small-feature
 ├── bug-fix
 └── improvement
```

### Used By

Google, Facebook, and organizations practicing Continuous Integration.

### Pros

* Fast integration
* Fewer merge conflicts
* Encourages CI/CD

### Cons

* Requires automated testing
* Demands discipline

---

## Answers

### Which strategy would you use for a startup shipping fast?

GitHub Flow because it is simple and supports rapid deployments.

### Which strategy would you use for a large team with scheduled releases?

GitFlow because it provides structured release management.

### Which strategy does an open-source project use?

Many GitHub open-source projects such as React primarily use a GitHub Flow style workflow with pull requests and feature branches.

---

# git reflog

## What is it?

Git Reflog records every movement of HEAD and branch references.

### Command

```bash
git reflog
```

### Why is it useful?

It can help recover commits that appear lost after a reset or rebase.

### Screenshot

> Insert screenshot of:
>
> ```bash
> git reflog
> ```

---

# Key Takeaways

* Learned the differences between soft, mixed, and hard resets.
* Practiced safely undoing commits with git revert.
* Compared reset and revert for different scenarios.
* Explored GitFlow, GitHub Flow, and Trunk-Based Development.
* Learned how git reflog can recover lost work.
* Improved understanding of real-world Git workflows.
