# Day 04 – Linux Practice: Processes and Services

## Process Checks

### 1. View running processes

```bash
ps aux
```

Output:
<img width="1314" height="780" alt="image" src="https://github.com/user-attachments/assets/5a1ca163-8455-4fba-8633-9ab5625c32a1" />




### 2. Monitor system processes

```bash
top
```

Observed active processes and CPU usage in real time.

Output:
<img width="1492" height="1080" alt="image" src="https://github.com/user-attachments/assets/da46d359-2d36-4466-829e-d919ecc8fa38" />


## Service Checks

### 3. Check SSH service status

```bash
systemctl status ssh
```

Output:
<img width="1348" height="606" alt="image" src="https://github.com/user-attachments/assets/82a2cd54-0a43-4e30-8a4b-9971d19957ab" />


### 4. List active services

```bash
systemctl list-units --type=service
```

Observed running services like ssh, cron, docker, and network manager.

Output:
<img width="1136" height="892" alt="image" src="https://github.com/user-attachments/assets/6e5f2e1c-1971-4fe8-9c14-e180fe84dd6a" />


---

## Log Checks

### 5. View logs for SSH service

```bash
journalctl -u ssh
```

Observed login activity and service startup logs.

Output:
<img width="1722" height="1154" alt="image" src="https://github.com/user-attachments/assets/54659b98-104e-4c53-b475-ec596d53752a" />


### 6. View latest system logs

```bash
tail -n 50 /var/log/syslog
```

Checked recent system log entries for troubleshooting.

Output:
<img width="3026" height="1246" alt="image" src="https://github.com/user-attachments/assets/f53a407d-42b0-41be-9177-f0f42d9959f6" />


---

## Mini Troubleshooting Steps

### Problem

SSH service was not responding initially.

### Troubleshooting Performed

Checked service status:

```bash
systemctl status ssh
```

Restarted SSH service:

```bash
sudo systemctl restart ssh
```

Verified service again:

```bash
systemctl status ssh
```

Confirmed the service was active and running successfully.

---

## What I Learned

* How to inspect running processes
* How to monitor system resources
* How to check Linux services using systemctl
* How to inspect logs using journalctl and tail
* Basic troubleshooting workflow for Linux services
