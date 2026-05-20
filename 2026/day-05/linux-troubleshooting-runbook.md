# Day 05 – Linux Troubleshooting Drill: CPU, Memory, and Logs

## Target Service / Process

Target service selected for troubleshooting drill:

```bash
ssh
```

Purpose:
Monitoring SSH service health, logs, CPU usage, memory usage, and connectivity.

---

# Environment Basics

## 1. Check Kernel and System Information

```bash
uname -a
```
Output:
<img width="1132" height="170" alt="image" src="https://github.com/user-attachments/assets/a54c5e16-4f62-4750-9ec6-a54049f1dddc" />


Observation:
Verified Linux kernel version and system architecture.

---

## 2. Check OS Release Information

```bash
cat /etc/os-release
```

Output:
<img width="1116" height="532" alt="image" src="https://github.com/user-attachments/assets/4bfccbc1-2060-433a-8c04-23af70340fe0" />


Observation:
Confirmed operating system version and distribution details.

---

# Filesystem Sanity Checks

## 3. Create Test Directory

```bash
mkdir /tmp/runbook-demo
```

Observation:
Created temporary directory successfully.

---

## 4. Copy and Verify File

```bash
cp /etc/hosts /tmp/runbook-demo/hosts-copy
ls -l /tmp/runbook-demo
```

Output:
<img width="1078" height="256" alt="image" src="https://github.com/user-attachments/assets/97670914-e8a2-4ee0-958b-55499014aecf" />


Observation:
Verified file copy operation and permissions.

---

# Snapshot: CPU & Memory

## 5. Monitor Processes

```bash
top
```
Output:
<img width="1126" height="824" alt="image" src="https://github.com/user-attachments/assets/4b1f1810-f105-4e83-abab-86e4dbd1a8a9" />


Observation:
CPU usage remained stable. No abnormal spikes observed during monitoring.

---

## 6. Check Memory Usage

```bash
free -h
```

Output:
<img width="1130" height="214" alt="image" src="https://github.com/user-attachments/assets/278dc435-1c9b-49ca-903e-1a4d0330a684" />



Observation:
Memory usage within healthy limits. No swap pressure detected.

---

# Snapshot: Disk & IO

## 7. Check Disk Usage

```bash
df -h
```

Output:
<img width="1134" height="608" alt="image" src="https://github.com/user-attachments/assets/570e51ec-c2ee-4047-b788-cc39c83a1a1e" />


Observation:
Sufficient free disk space available.

---

## 8. Check Log Directory Size

```bash
du -sh /var/log
```

Output:
<img width="688" height="122" alt="image" src="https://github.com/user-attachments/assets/4106c95a-66df-4887-a274-02a770f26099" />


Observation:
Log directory size normal. No excessive log growth found.

---

# Snapshot: Network

## 9. Check Listening Ports

```bash
ss -tulpn
```
Output:
<img width="1122" height="498" alt="image" src="https://github.com/user-attachments/assets/e9252850-1b7f-4499-a42b-335b2ea4d226" />


Observation:
SSH service listening correctly on port 22.

---

## 10. Test Network Connectivity

```bash
curl -I https://google.com
```

Output:
<img width="1136" height="560" alt="image" src="https://github.com/user-attachments/assets/1579f81a-7338-47a8-8e4c-14e6deb5a62f" />


Observation:
Outbound internet connectivity working successfully.

---

# Logs Reviewed

## 11. Review SSH Service Logs

```bash
journalctl -u ssh -n 50
```

Output:
<img width="1128" height="780" alt="image" src="https://github.com/user-attachments/assets/14d063cd-25b6-499c-9a66-24260a9d0e76" />


Observation:
No recent authentication failures or service crashes observed.

---

## 12. Review System Logs

```bash
tail -n 50 /var/log/syslog
```

Output:
<img width="1138" height="838" alt="image" src="https://github.com/user-attachments/assets/6667053a-cecd-46b2-a420-8d3134551ab7" />


Observation:
System logs appeared clean with no critical errors.

---

# Quick Findings

* SSH service running normally
* CPU and memory usage stable
* Disk utilization healthy
* No major log errors detected
* Network connectivity functioning correctly

---

# If This Worsens (Next Steps)

1. Restart SSH service safely:

```bash
sudo systemctl restart ssh
```

2. Increase log inspection and monitor live logs:

```bash
journalctl -u ssh -f
```

3. Collect deeper diagnostics using:

```bash
strace
vmstat
iostat
```

---

# What I Learned

* How to collect system health information quickly
* How to inspect services and logs systematically
* How to verify CPU, memory, disk, and network status
* Importance of capturing evidence before restarting services
