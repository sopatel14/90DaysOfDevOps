# Day 24 – Advanced Git: Merge, Rebase, Stash & Cherry Pick

## Objective

Learn how Git combines branches, rewrites commit history, temporarily saves work, and selectively applies commits.

---

# Task 1: Git Merge

## Fast-Forward Merge

A fast-forward merge occurs when the target branch has not moved ahead since the feature branch was created. Git simply moves the branch pointer forward without creating an additional merge commit.

### Example Commands

```bash
git checkout main
git checkout -b feature-login
```

**What it does:** Creates and switches to a new branch called `feature-login`.

```bash
git commit -m "Add login page"
```

**What it does:** Creates a new commit containing staged changes.

```bash
git checkout main
```

**What it does:** Switches back to the main branch.

```bash
git merge feature-login
```

**What it does:** Merges changes from `feature-login` into `main`.

### Observation

Git performed a fast-forward merge because `main` had no additional commits.

### Output

<img width="1220" height="790" alt="image" src="https://github.com/user-attachments/assets/9db544cd-b6a7-444d-a34f-1d80d3b14389" />


---

## Merge Commit

A merge commit is created when both branches have new commits and Git needs to combine two separate histories.

### Example Commands

```bash
git checkout -b feature-signup
```

**What it does:** Creates a new feature branch.

```bash
git commit -m "Add signup page"
```

**What it does:** Saves changes on the feature branch.

```bash
git checkout main
git commit -m "Update main branch"
```

**What it does:** Creates a new commit on `main`.

```bash
git merge feature-signup
```

**What it does:** Combines changes from `feature-signup` into `main`.

### Observation

Git created a merge commit because both branches contained unique commits.

### Output

<img width="1478" height="1652" alt="image" src="https://github.com/user-attachments/assets/1d67a8fd-7f92-4528-a8d4-f40e0c0c69c8" />


---

## Merge Conflict

A merge conflict happens when Git cannot automatically determine which change should be kept.

### Example

The same line was modified in both branches.

### Conflict Markers

```text
<<<<<<< HEAD
Main Version
=======
Branch Version
>>>>>>> feature-branch
```

### Resolution

Edit the file manually, remove conflict markers, save the desired content, then commit the resolution.

### Output

<img width="1388" height="1450" alt="image" src="https://github.com/user-attachments/assets/74c4cb4a-892a-4d1e-9e26-4af0b791b41a" />


---

# Task 2: Git Rebase

## What is Rebase?

Rebase moves or reapplies commits from one branch onto another branch, creating a cleaner and more linear history.

### Example Commands

```bash
git checkout -b feature-dashboard
```

**What it does:** Creates the dashboard feature branch.

```bash
git commit -m "Add dashboard UI"
```

**What it does:** Saves dashboard-related changes.

```bash
git checkout main
git commit -m "Update main"
```

**What it does:** Advances the main branch.

```bash
git checkout feature-dashboard
git rebase main
```

**What it does:** Replays feature branch commits on top of the latest `main`.

### Observation

The commit history became linear and easier to read.

### Output

<img width="1570" height="2032" alt="image" src="https://github.com/user-attachments/assets/e55c0c9e-6e84-4271-9530-9293c7f2c4bb" />


---

## Answers

### What does rebase actually do to your commits?

Rebase reapplies existing commits onto a new base commit, effectively rewriting commit history.

### How is the history different from a merge?

Merge preserves branch history and adds a merge commit. Rebase creates a cleaner, linear history without merge commits.

### Why should you never rebase shared commits?

Because rebase changes commit hashes and can cause confusion or conflicts for other team members.

### When would you use rebase vs merge?

* Use rebase to keep history clean before merging.
* Use merge when preserving branch history is important.

---

# Task 3: Squash Merge vs Regular Merge

## Squash Merge

A squash merge combines multiple commits into a single commit before merging.

### Example Commands

```bash
git merge --squash feature-profile
```

**What it does:** Combines all feature branch commits into one staged change.

```bash
git commit -m "Add profile feature"
```

**What it does:** Creates a single commit containing all squashed changes.

### Observation

Only one commit was added to `main`.

### Output

<img width="1110" height="2040" alt="image" src="https://github.com/user-attachments/assets/b3076541-70e0-4ba9-8dde-38d15fef33e5" />


---

## Regular Merge

```bash
git merge feature-settings
```

**What it does:** Merges the branch while preserving individual commits.

### Observation

All commits from the feature branch remained visible in Git history.

### History

<img width="1660" height="1942" alt="image" src="https://github.com/user-attachments/assets/928e07ba-b607-4ef4-b58b-0261bd086930" />


---

## Answers

### What does squash merging do?

Combines multiple commits into a single commit before merging.

### When would you use squash merge?

When a feature branch contains many small or temporary commits.

### When would you use a regular merge?

When you want to preserve the complete development history.

### What is the trade-off of squashing?

You lose detailed commit history from the feature branch.

---

# Task 4: Git Stash

## What is Git Stash?

Git Stash temporarily saves uncommitted changes so you can switch branches without committing incomplete work.

### Example Commands

```bash
git stash
```

**What it does:** Saves current uncommitted changes and cleans the working directory.

```bash
git stash push -m "login work"
```

**What it does:** Saves changes with a descriptive message.

```bash
git stash list
```

**What it does:** Displays all saved stashes.

```bash
git stash pop
```

**What it does:** Applies the latest stash and removes it from the stash list.

```bash
git stash apply
```

**What it does:** Applies a stash without removing it.

```bash
git stash apply stash@{0}
```

**What it does:** Applies a specific stash entry.

### Screenshot

<img width="1316" height="1792" alt="image" src="https://github.com/user-attachments/assets/e7b8e7d0-b534-4c61-9189-066ea9d5e78d" />


---

## Answers

### Difference between git stash pop and git stash apply?

* `git stash pop` applies and removes the stash.
* `git stash apply` applies but keeps the stash saved.

### When would you use stash in a real-world workflow?

When urgent work interrupts current development and changes are not ready to commit.

---

# Task 5: Cherry Pick

## What is Cherry Pick?

Cherry-pick copies a specific commit from one branch and applies it to another branch.

### Example Commands

```bash
git log --oneline
```

**What it does:** Displays commit history with commit hashes.

```bash
git cherry-pick <commit-hash>
```

**What it does:** Applies a specific commit to the current branch.

### Observation

Only the selected commit was copied to `main`.

### Output

<img width="2356" height="2100" alt="image" src="https://github.com/user-attachments/assets/c5c8d83d-129a-4fc6-8946-27d3a3dfd911" />


---

## Answers

### What does cherry-pick do?

Copies and applies a specific commit from one branch to another.

### When would you use cherry-pick in a real project?

For hotfixes, bug fixes, or selectively moving changes without merging an entire branch.

### What can go wrong with cherry-picking?

* Merge conflicts may occur.
* Duplicate commits can appear.
* Commit history can become harder to understand if overused.

---
