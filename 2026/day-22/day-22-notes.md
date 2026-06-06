# Day 22 Notes - Introduction to Git

## What is the difference between git add and git commit?

`git add` places changes into the staging area.
`git commit` permanently records the staged changes in Git history.

<img width="1020" height="406" alt="image" src="https://github.com/user-attachments/assets/5bf23ddb-6501-443d-816f-daeb0ba5585a" />

<img width="1128" height="748" alt="image" src="https://github.com/user-attachments/assets/075b7053-7c49-4f2b-be2f-3f14c1ba9f7e" />


---

## What does the staging area do? Why doesn't Git just commit directly?

The staging area acts like a review area where I can choose exactly which changes will be included in the next commit.

Git doesn't commit directly because developers often make multiple changes and may only want to commit some of them.

---

## What information does git log show you?

`git log` shows:

<img width="1046" height="682" alt="image" src="https://github.com/user-attachments/assets/985cd8cc-ee37-420b-99ab-a376077c2246" />



* Commit ID (hash)
* Author
* Date and time
* Commit message

---

## What is the .git/ folder and what happens if you delete it?

The `.git/` folder contains all repository metadata, commit history, branches, and configuration.


<img width="522" height="89" alt="image" src="https://github.com/user-attachments/assets/0cf000ed-9f80-4103-b350-ce7e65c7cf6e" />


If I delete `.git/`, the project files remain but Git tracking and history are lost.

---

## What is the difference between a working directory, staging area, and repository?

### Working Directory

The actual files I'm editing.

### Staging Area

A temporary area where changes are prepared for a commit.

### Repository

The Git database that stores commits and project history.



