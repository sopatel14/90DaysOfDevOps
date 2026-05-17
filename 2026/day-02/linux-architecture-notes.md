# Linux Architecture Notes

## 1. Core Components of Linux

**User Space**
- The area where user applications run.
- Examples:
  - Shell (bash)
  - Terminal
  - Nginx
  - Docker
  - Jenkins
- Applications cannot access hardware directly.
- They communicate with the kernel using **system calls**.

 **Shell**
- A command-line interface between the user and kernel.
- Interprets commands and sends requests to the kernel.
- Examples:
  - bash
  - zsh
  - sh
 
**Kernel**
- The **Kernel** is the core of Linux.
- It directly interacts with hardware (CPU, memory, disk, network).
- Written mainly in **C language**.
- Responsibilities:
  - Process management
  - Memory management
  - Device management
  - CPU scheduling
  - File systems

### Init / systemd
- The first process started during boot.
- Runs with **PID 1**.
- Responsible for starting and managing services.
- Modern Linux distributions use **systemd** as the init system.

---

## 2. Process Creation and Management

### What is a Process?
- A process is a running instance of a program.
- Every process has:
  - PID (Process ID)
  - Parent Process
  - Memory allocation
  - CPU usage

### Process Lifecycle
1. Program starts
2. Kernel creates a process
3. Scheduler gives CPU time
4. Process waits/runs/stops
5. Process exits

### Process States
- **Running** → Currently using CPU
- **Sleeping/Waiting** → Waiting for I/O or resources
- **Stopped** → Paused by signal/user
- **Zombie** → Process finished but parent has not collected exit status

### CPU Scheduling
- Kernel scheduler decides which process gets CPU time.
- Based on:
  - Priority
  - Fairness
  - Resource availability

---

## 3. What systemd Does

### systemd Overview
- systemd is the service manager in modern Linux systems.
- Starts services during boot.
- Monitors and restarts failed services.

### Why systemd Matters for DevOps
- Helps manage production services reliably.
- Makes troubleshooting easier.
- Provides centralized logging and service control.

### Common systemd Tasks
```bash
systemctl status nginx
systemctl restart docker
systemctl stop jenkins
systemctl enable nginx



4. Daily Linux Commands for DevOps
grep

Search text or logs for patterns.

grep -i "error" /var/log/syslog

systemctl : Manage Linux services.

systemctl status nginx


top / htop : Monitor CPU, memory, and running processes.

htop


tail : View live logs in real time.

tail -f /var/log/nginx/access.log

df : Check disk usage.

df -h
