# Day 23 Notes – Git Branching & Working with GitHub

## What is a branch in Git?

A branch is an independent line of development that allows changes to be made without affecting other branches.

---

## Why do we use branches instead of committing everything to main?

Branches allow developers to work on features, bug fixes, and experiments safely without impacting the stable main branch.

---

## What is HEAD in Git?

HEAD is a pointer that references the current commit and branch that I am currently working on.

---

## What happens to your files when you switch branches?

Git updates the working directory to match the selected branch. Files may appear, disappear, or change depending on the contents of that branch.

---

## Branching Commands – Hands-On

### Tasks Completed

* Listed all branches in the repository.
* Created a new branch called `feature-1`.
* Switched to `feature-1`.
* Created and switched to `feature-2` using a single command.
* Practiced moving between branches using `git switch`.
* Created a commit on `feature-1`.
* Verified that the commit did not exist on `main`.
* Deleted an unused branch.
* Updated `git-commands.md` with branching commands.

### Evidence

**Screenshot 1:** Branch creation and switching

<img width="1138" height="330" alt="image" src="https://github.com/user-attachments/assets/1f8b8b2c-198e-4ddc-bf3c-8e33fba40594" />

<img width="1214" height="178" alt="image" src="https://github.com/user-attachments/assets/346cec4d-aa3b-44dc-8c8d-f80addf0bb54" />



**Screenshot 2:** Commit created on `feature-1`

<img width="1334" height="494" alt="image" src="https://github.com/user-attachments/assets/d79cf089-af7c-4c92-a7fe-4306172f1477" />


**Screenshot 3:** Verification that commit is absent from `main`

<img width="1190" height="264" alt="image" src="https://github.com/user-attachments/assets/36346569-ef33-4c57-8534-4a0b0b4b1d46" />

**Screenshot 4:** Delete an unused branch

<img width="1164" height="162" alt="image" src="https://github.com/user-attachments/assets/f56034dd-e64c-4e73-93cc-0ca24460ec70" />



---

## Push to GitHub

### Tasks Completed

* Created a GitHub repository.
* Connected local repository to remote.
* Pushed `main` branch.
* Pushed `feature-1` branch.
* Verified both branches on GitHub.

### Evidence

**Screenshot 5:** GitHub repository showing branches

<img width="417" height="463" alt="image" src="https://github.com/user-attachments/assets/ff9a3477-dd16-44f8-b1a0-1fd6fc745cbe" />


---

## What is the difference between origin and upstream?

`origin` is the remote repository that I own and push changes to.

`upstream` is the original repository from which a fork was created.

---

## Pull from GitHub

### Tasks Completed

* Edited a file directly on GitHub.
* Pulled the changes into the local repository.

### Evidence


**Screenshot 6:** Successful git pull

<img width="1256" height="690" alt="image" src="https://github.com/user-attachments/assets/f0b72ddf-7e58-444d-9728-9555f8a33feb" />


---

## What is the difference between git fetch and git pull?

`git fetch` downloads changes from the remote repository but does not merge them.

`git pull` downloads changes and automatically merges them into the current branch.

---

## Clone vs Fork

### What is the difference between clone and fork?

A clone creates a local copy of a repository on my machine.

A fork creates a copy of a repository under my own GitHub account.

---

### When would you clone vs fork?

I would clone repositories that I have direct access to.

I would fork repositories when contributing to projects owned by other people or organizations.

---

### After forking, how do you keep your fork in sync with the original repository?

By adding the original repository as an `upstream` remote and periodically fetching and merging changes from it.
