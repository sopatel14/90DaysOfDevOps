# Day 12 – Revision & Breather (Days 01–11)

## Goal
Revise Linux and DevOps fundamentals learned in Days 01–11 and strengthen weak areas through quick hands-on practice.

---

# 1. Mindset & Learning Plan Review

## Original Goal
- Learn Linux fundamentals
- Become comfortable with terminal usage
- Build DevOps basics step-by-step
- Improve troubleshooting confidence

## Updates After 11 Days
- Linux commands now feel more familiar
- Need more practice with:
  - chmod numeric permissions
  - systemctl & journalctl
  - file ownership concepts
- Confidence improving with daily practice

## Next Goal
- Focus more on:
  - Services troubleshooting
  - User/group management
  - File permissions
  - Real Linux scenarios

---

# 2. Processes & Services Revision

## Command 1

```bash
ps aux
```

### Observation
- Shows all running processes
- Helps identify CPU/memory usage
- Useful during troubleshooting

---

## Command 2

```bash
systemctl status ssh
```

### Observation
- Checks if SSH service is active
- Shows service state and logs
- Useful for verifying server connectivity

---

## Optional Log Check

```bash
journalctl -u ssh
```

### Observation
- Displays logs related to SSH service
- Helpful for debugging login/service issues

---

# 3. File Skills Practice

## Create and Append Text

```bash
echo "DevOps Practice" >> notes.txt
```

### Learned
- Appends text safely without overwriting file

---

## Change Permissions

```bash
chmod 755 script.sh
```

### Learned
- Gives owner full access
- Others get read + execute access

---

## Check File Permissions

```bash
ls -l
```

### Learned
- Displays permissions, ownership, and file details

---

# 4. Cheat Sheet Refresh – Top 5 Useful Commands

## 1. List Files with Details

```bash
ls -l
```

Used to check files, permissions, and ownership quickly.

---

## 2. Print Working Directory

```bash
pwd
```

Helps confirm current working directory.

---

## 3. Change Directory

```bash
cd
```

Used constantly for navigation.

---

## 4. Check Running Processes

```bash
ps aux
```

Useful for process troubleshooting.

---

## 5. Check Service Status

```bash
systemctl status <service>
```

Checks service health immediately.

---

# 5. User & Group Sanity Check

## Create User

```bash
sudo useradd devuser
```

---

## Verify User

```bash
id devuser
```

---

## Change Ownership

```bash
sudo chown devuser notes.txt
```

---

## Verify Ownership

```bash
ls -l notes.txt
```

### Learned
- Ownership changes affect file access
- Always verify changes using ls -l

---

# 6. Mini Self-Check

## Q1. Which 3 commands save you the most time right now, and why?

### 1. Check File Permissions

```bash
ls -ld devops
```

- Quickly shows permissions, ownership, and directory details
- Helps verify access issues immediately

---

### 2. Check Service Health

```bash
systemctl status docker
```

- Instantly tells whether Docker service is running or failed
- Useful during troubleshooting

---

### 3. Enable Docker at Boot

```bash
systemctl enable docker
```

- Automatically starts Docker whenever the EC2 instance reboots
- Saves time because Docker starts without manual intervention every session

## Q2. How do you check if a service is healthy?

### Commands

```bash
systemctl status nginx
```

```bash
ps aux | grep nginx
```

```bash
journalctl -u nginx
```

### Why?
- Confirms service state
- Verifies process is running
- Checks logs for errors

---

## Q3. How do you safely change ownership and permissions?

### Example Commands

```bash
sudo chown devuser:devuser file.txt
```

```bash
chmod 644 file.txt
```

### Verify Changes

```bash
ls -l file.txt
```

### Notes
- Avoid using 777 permissions
- Always verify ownership and permissions after changes

---

## Q4. What will you focus on improving in the next 3 days?

### Focus Areas
- More Linux troubleshooting
- Faster terminal navigation
- Better understanding of permissions
- Practice real-world service debugging

---

# 7. Key Takeaways

- Consistency matters more than speed
- Linux basics are becoming easier daily
- Troubleshooting requires observation + logs
- File permissions and ownership are critical in DevOps
- Small daily practice builds confidence

---

# Revision Complete ✅
