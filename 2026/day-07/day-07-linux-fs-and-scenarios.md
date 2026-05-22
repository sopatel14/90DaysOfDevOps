Day 07 – Linux File System Hierarchy and Scenario Practice

Part 1 – Linux File System Hierarchy

1. Root Directory (/)

Purpose:
The root directory is the starting point of the Linux filesystem. All files and directories originate from here.

Command:
```bash
ls -l /
```

Output:

<img width="474" height="412" alt="image" src="https://github.com/user-attachments/assets/3b7fc5c9-7765-4687-b9d7-3fd807a4fcb7" />



I would use this when:
Navigating the Linux filesystem structure or troubleshooting system-level issues.

2. Home Directory (/home)

Purpose:
Contains personal directories for regular users.

Command:

```bash
ls -l /home
```

Output:

<img width="390" height="76" alt="image" src="https://github.com/user-attachments/assets/bfd9d3ac-4ccc-45c3-8f8a-782c2aac0a38" />


I would use this when:
Managing user files and personal project directories.

3. Root User Directory (/root)

Purpose:
Home directory for the root administrator account.

Command:

```bash
ls -l /root
```

Output :

<img width="416" height="109" alt="image" src="https://github.com/user-attachments/assets/d4fa1b13-08e6-4074-a18f-3baf5e14aebb" />


I would use this when:
Performing administrative or privileged tasks.

4. Configuration Directory (/etc)

Purpose:
Stores system-wide configuration files and service settings.

Command:

```bash
ls -l /etc
```
Output:

<img width="522" height="779" alt="image" src="https://github.com/user-attachments/assets/3d5f15a1-023e-4b74-82bc-0c22276eafbd" />


I would use this when:
Editing configurations for services, networking, or system behavior.

5. Log Directory (/var/log)

Purpose:
Contains system logs and application logs used for troubleshooting.

Command:

```bash
ls -l /var/log
```

Output:

<img width="568" height="427" alt="image" src="https://github.com/user-attachments/assets/50d1923c-175d-4fa7-91bf-83878e84cde4" />


I would use this when:
Investigating service failures, login issues, or system errors.

6. Temporary Files Directory (/tmp)

Purpose:
Stores temporary files created by applications and users.

Command:

```bash
ls -l /tmp
```

Output:

<img width="567" height="252" alt="image" src="https://github.com/user-attachments/assets/53d84f82-1296-4641-af33-dbf2d42461e7" />


I would use this when:
Testing scripts or storing temporary troubleshooting files.

Additional Directories
7. Binary Commands Directory (/bin)

Purpose:
Contains essential Linux command binaries.

Command:

```bash
cd /bin
ls
```

Output:

<img width="444" height="76" alt="image" src="https://github.com/user-attachments/assets/9a233aa0-cdf7-47ec-bb90-a162b6f858d5" />


I would use this when:
Running essential Linux commands required for system operation.


I would use this when:
Checking installed tools and command locations.

8. Optional Applications Directory (/opt)

Purpose:
Stores third-party or optional software applications.

Command:

```bash
ls -l /opt
```

Output:

<img width="318" height="74" alt="image" src="https://github.com/user-attachments/assets/93a6320d-4e78-4bce-9479-6154f9457d82" />


I would use this when:
Managing manually installed applications or enterprise software.

Hands-on Tasks

Find Largest Log Files

Command:

```bash
du -sh /var/log/* 2>/dev/null | sort -h | tail -5
```

Output:

<img width="551" height="143" alt="image" src="https://github.com/user-attachments/assets/f8606260-4523-4c62-b1a2-8d9a3892dc9d" />

View Hostname Configuration

Command:

```bash
cat /etc/hostname
whoami
```

Output:

<img width="355" height="92" alt="image" src="https://github.com/user-attachments/assets/e5d6e5e4-af60-4d33-afaf-681091b001da" />


Observation:
Verified the system hostname configuration.

Check Home Directory

Command:

```bash
ls -la ~
```

Output:

<img width="454" height="301" alt="image" src="https://github.com/user-attachments/assets/61ec1a94-5650-4b85-b3ee-75412820fc36" />


Observation:
Reviewed hidden files, configuration files, and user directories.



Part 2: Scenario-Based Practice

Scenario 1: Service Not Starting

Question

A web application service called myapp failed to start after a server reboot.
What commands would you run to diagnose the issue?

Step 1: Check service status

```bash
systemctl status myapp
```

Why: Checks whether the service is active, failed, or stopped and shows recent errors.

Step 2: Check service logs

```bash
journalctl -u myapp -n 50
```

Why: Displays the last 50 log lines for troubleshooting startup issues.

Step 3: Verify if the service is enabled on boot

```bash
systemctl is-enabled myapp
```

Why: Confirms whether the service is configured to start automatically after reboot.

Step 4: Check failed services

```bash
systemctl --failed
```

Why: Shows all services that failed during boot.

Step 5: Restart the service manually

```bash
systemctl restart myapp
```

Why: Attempts to start the service again and generate fresh logs.

What I Learned

Always start by checking the service status, then inspect logs, verify boot settings, and test the service again.

Scenario 2: High CPU Usage

Question

Your manager reports that the application server is slow.
You SSH into the server. What commands would you run to identify which process is using high CPU?

Step 1: Monitor live CPU usage

```bash
top
```
Output:

<img width="566" height="424" alt="image" src="https://github.com/user-attachments/assets/3b1048d8-6c59-4286-b812-39fd376ececb" />


Why: Shows real-time CPU and memory usage of running processes.

Step 2: List processes sorted by CPU usage

```bash
ps aux --sort=-%cpu | head -10
```
Output:

<img width="566" height="371" alt="image" src="https://github.com/user-attachments/assets/fcad5684-c74a-4417-9e6c-b7fd82706476" />


Why: Displays the top CPU-consuming processes along with their PID.

Step 3: Use enhanced interactive monitoring

```bash
htop
```

Output:

<img width="565" height="419" alt="image" src="https://github.com/user-attachments/assets/b82c63c6-fc97-4461-879a-7d0aeefd61c4" />


Why: Provides an easier and more user-friendly process monitoring interface.

Step 4: Inspect a specific process

```bash
ps -p <PID> -f
```

Output:

<img width="435" height="107" alt="image" src="https://github.com/user-attachments/assets/72b5b3c1-2296-40ba-98b9-b88359e2e101" />


Why: Displays detailed information about a specific high-CPU process.

What I Learned

Use monitoring tools like top or htop first, identify the PID of the high CPU process, and then inspect it further.

Scenario 3: Finding Service Logs
Question

A developer asks: “Where are the logs for the docker service?”
The service is managed by systemd.

Step 1: Check service status

```bash
systemctl status docker
```

Output:

<img width="665" height="423" alt="image" src="https://github.com/user-attachments/assets/ecd25110-6bd4-47f5-be51-5e1438a74aad" />


Why: Shows whether Docker is running and displays recent logs.

Step 2: View recent logs

```bash
journalctl -u docker -n 50
```
Output:

<img width="661" height="360" alt="image" src="https://github.com/user-attachments/assets/9443d967-c552-415f-a5f5-b45be0c7635e" />



Why: Displays the last 50 lines of Docker service logs.

Step 3: Follow logs in real-time

```bash
journalctl -u docker -f
```
Output:

<img width="663" height="392" alt="image" src="https://github.com/user-attachments/assets/5795f2ad-06ce-4ae9-a801-a133a5a2a954" />


Why: Streams live logs continuously like tail -f.

Step 4: View logs since last boot

```bash
journalctl -u docker -b
```

Output:

<img width="663" height="360" alt="image" src="https://github.com/user-attachments/assets/a6d329d9-e565-49ea-921e-b22682134f9a" />


Why: Helps troubleshoot issues that occurred after reboot.

What I Learned

Systemd-managed services store logs in journald, and journalctl is the main command used for viewing and monitoring logs.

Scenario 4: File Permissions Issue
Question

A script at /home/ubuntu/backup.sh is not executing.

When running:

```bash
./backup.sh
```

You get:

<img width="348" height="75" alt="image" src="https://github.com/user-attachments/assets/39c63b5b-9ae5-441c-8798-e62f3c63770b" />


Permission denied

What commands would you use to fix this?

Step 1: Check current permissions

```bash
ls -l /home/ubuntu/backup.sh
```

Example output:

<img width="490" height="68" alt="image" src="https://github.com/user-attachments/assets/85c546bd-8707-48ad-b2d1-9d811ee9bc8c" />


Why: Verifies whether the script has execute (x) permission.


Step 2: Add execute permission

```bash
chmod +x /home/ubuntu/backup.sh
```

Why: Makes the script executable.

Step 3: Verify updated permissions

```bash
ls -l /home/ubuntu/backup.sh
```
Example output:

<img width="526" height="58" alt="image" src="https://github.com/user-attachments/assets/c6533581-fc03-4c66-b4a3-5b6c9e570414" />


Why: Confirms that execute permission was added successfully.


Step 4: Run the script again

```bash
./backup.sh
```
Example Output:

<img width="301" height="58" alt="image" src="https://github.com/user-attachments/assets/cdcae6e8-f89f-4e32-b725-e1350c6ea6c0" />


Why: Tests whether the permission issue is resolved.

## What I Learned

Linux files require execute (`x`) permission to run scripts.  
Use `chmod +x` to make scripts executable.

---

# Key Troubleshooting Flow

1. Check the current status of the service or process  
2. Review logs for error messages  
3. Verify permissions or configuration  
4. Restart or test again after changes  
5. Confirm the issue is resolved  

---

# Why This Matters for DevOps

These troubleshooting skills are important for:

- Diagnosing production issues
- Managing Linux servers
- Monitoring applications and services
- Debugging deployment failures
- Handling real-world DevOps incidents
- Performing well in technical interviews
