# Day 28 – Revision Day (Days 1–27)

## Self-Assessment Checklist

### Linux

| Topic                                          | Status               |
| ---------------------------------------------- | -------------------- |
| Navigate file system, create/move/delete files | ✅ Can do confidently |
| Manage processes (ps, top, kill, jobs, fg, bg) | ✅ Can do confidently |
| Work with systemd services                     | ✅ Can do confidently |
| Edit files using vim/nano                      | ✅ Can do confidently |
| CPU, memory, disk troubleshooting              | ✅ Can do confidently |
| Linux file system hierarchy                    | ✅ Can do confidently |
| User and group management                      | ✅ Can do confidently |
| File permissions (chmod)                       | ✅ Can do confidently |
| Ownership management (chown/chgrp)             | ✅ Can do confidently |
| LVM management                                 | ⚠️ Need to revisit   |
| Network troubleshooting tools                  | ✅ Can do confidently |
| DNS, IPs, subnets, ports                       | ⚠️ Need to revisit   |

---

### Shell Scripting

| Topic                                    | Status               |
| ---------------------------------------- | -------------------- |
| Variables, arguments, user input         | ✅ Can do confidently |
| if/elif/else and case statements         | ✅ Can do confidently |
| Loops (for, while, until)                | ✅ Can do confidently |
| Functions                                | ✅ Can do confidently |
| grep, awk, sed, sort, uniq               | ⚠️ Need to revisit   |
| Error handling (set -euo pipefail, trap) | ✅ Can do confidently |
| Crontab scheduling                       | ✅ Can do confidently |

---

### Git & GitHub

| Topic                  | Status               |
| ---------------------- | -------------------- |
| Init, add, commit, log | ✅ Can do confidently |
| Branching              | ✅ Can do confidently |
| Push and pull          | ✅ Can do confidently |
| Clone vs Fork          | ✅ Can do confidently |
| Merge strategies       | ✅ Can do confidently |
| Rebase                 | ⚠️ Need to revisit   |
| Git stash              | ✅ Can do confidently |
| Cherry-pick            | ✅ Can do confidently |
| Squash merge           | ⚠️ Need to revisit   |
| Reset and Revert       | ✅ Can do confidently |
| Branching strategies   | ⚠️ Need to revisit   |
| GitHub CLI             | ✅ Can do confidently |

---

## Topics Revisited

### 1. LVM (Logical Volume Manager)

Re-learned:

* Physical Volumes (PV) are created from disks.
* Volume Groups (VG) combine one or more PVs.
* Logical Volumes (LV) are created from VGs.
* LVM allows resizing storage without repartitioning.

Useful commands:

```bash
sudo pvcreate /dev/xvdf
sudo vgcreate my_vg /dev/xvdf
sudo lvcreate -L 5G -n my_lv my_vg
sudo lvextend -L +1G /dev/my_vg/my_lv
```

---

### 2. DNS & Networking

Re-learned:

* DNS translates domain names into IP addresses.
* Port 80 = HTTP
* Port 443 = HTTPS
* Port 22 = SSH
* Port 53 = DNS
* Tools: ping, curl, dig, nslookup, ss

Examples:

```bash
ping google.com
curl https://google.com
dig google.com
nslookup google.com
ss -tulpn
```

---

### 3. Git Rebase & Branching Strategies

Re-learned:

* Rebase rewrites commit history to create a cleaner timeline.
* Merge preserves branch history.
* GitHub Flow is good for small teams deploying frequently.
* GitFlow is useful for large projects with release cycles.
* Trunk-Based Development encourages short-lived branches.

Example:

```bash
git checkout feature-1
git rebase main
```

---

# Quick-Fire Answers

### What does chmod 755 script.sh do?

Gives:

* Owner: read, write, execute
* Group: read, execute
* Others: read, execute

Makes the script executable while keeping write access only for the owner.

---

### What is the difference between a process and a service?

A process is a running program.

A service is a background process managed by the operating system (usually through systemd).

---

### How do you find which process is using port 8080?

```bash
sudo ss -tulpn | grep 8080
```

or

```bash
sudo lsof -i :8080
```

---

### What does set -euo pipefail do?

```bash
set -e
```

Exit when a command fails.

```bash
set -u
```

Treat unset variables as errors.

```bash
set -o pipefail
```

Fail if any command inside a pipeline fails.

---

### What is the difference between git reset --hard and git revert?

`git reset --hard`

* Removes commits locally.
* Rewrites history.

`git revert`

* Creates a new commit that undoes changes.
* Safe for shared branches.

---

### What branching strategy would you recommend for a team of 5 developers shipping weekly?

GitHub Flow.

Reason:

* Simple workflow
* Feature branches
* Pull requests
* Fast releases

---

### What does git stash do and when would you use it?

Temporarily saves uncommitted changes so you can switch branches or pull updates without committing incomplete work.

```bash
git stash
git stash pop
```

---

### How do you schedule a script to run every day at 3 AM?

```bash
crontab -e
```

Add:

```bash
0 3 * * * /home/ubuntu/scripts/backup.sh
```

---

### What is the difference between git fetch and git pull?

```bash
git fetch
```

Downloads changes without merging.

```bash
git pull
```

Downloads and merges changes into the current branch.

---

### What is LVM and why would you use it instead of regular partitions?

LVM (Logical Volume Manager) provides flexible storage management.

Benefits:

* Easy resizing
* Multiple disks in one volume group
* Snapshots
* Better scalability than fixed partitions

---

# Repository Health Check

Completed:

* All Day 1–27 submissions pushed to GitHub
* Git commands notes updated
* Shell scripting cheat sheet verified
* GitHub profile README reviewed
* Repository structure organized

---

# Teach It Back

## Explaining Git Branches to a Beginner

Think of a Git branch as a separate workspace where you can make changes without affecting the main project. Developers create feature branches to work on new functionality safely. Once the feature is tested and ready, it is merged back into the main branch. This allows multiple developers to work on different features simultaneously without overwriting each other's work. Branching is one of Git's most powerful features because it makes collaboration safer and more organized.

---

## Key Takeaways from Days 1–27

* Built strong Linux administration fundamentals.
* Learned networking concepts required for DevOps.
* Automated tasks using Shell scripting.
* Practiced real-world scripting projects.
* Mastered Git workflows and collaboration concepts.
* Improved GitHub profile and developer branding.
* Developed confidence working from the Linux terminal.

Day 28 reinforced the importance of revision and identifying weak areas before moving on to advanced DevOps topics.
