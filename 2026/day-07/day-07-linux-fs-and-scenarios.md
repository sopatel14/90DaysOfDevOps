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

