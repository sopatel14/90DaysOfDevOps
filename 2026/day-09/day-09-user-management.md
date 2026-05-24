# Day 09 – Linux User & Group Management Challenge

## Task 1: Create Users

### Create Users with Home Directories

```bash
sudo useradd -m tokyo
sudo useradd -m berlin
sudo useradd -m professor
```
Output :

<img width="471" height="91" alt="image" src="https://github.com/user-attachments/assets/795f32a0-db31-4d27-9825-4e6a3a078203" />



### Set Passwords

```bash
sudo passwd tokyo
sudo passwd berlin
sudo passwd professor
```
Output:

<img width="443" height="284" alt="image" src="https://github.com/user-attachments/assets/321170f6-c5ea-4666-9b82-4174ed56bf56" />


### Verify Users

Check `/etc/passwd`:

```bash
cat /etc/passwd | grep -E 'tokyo|berlin|professor'
```
Output:

<img width="562" height="103" alt="image" src="https://github.com/user-attachments/assets/49f72af4-29cc-4d12-a368-fa2e82974e70" />







Check home directories:

```bash
ls /home
```
Output:

<img width="361" height="88" alt="image" src="https://github.com/user-attachments/assets/2f97b5e2-3cbb-47fd-9112-6f507f670088" />


---


# Task 2: Create Groups

### Create Groups

```bash
sudo groupadd developers
sudo groupadd admins
```
Output:

<img width="453" height="88" alt="image" src="https://github.com/user-attachments/assets/09879f72-8fe0-40f7-91b6-4f72ec3edb84" />


### Verify Groups

```bash
cat /etc/group | grep -E 'developers|admins'
```

Output :

<img width="554" height="117" alt="image" src="https://github.com/user-attachments/assets/51f8db87-69ac-4299-bc43-d211404fe73b" />


---

# Task 3: Assign Users to Groups

### Add Users to Groups

```bash
sudo usermod -aG developers tokyo

sudo usermod -aG developers,admins berlin

sudo usermod -aG admins professor
```

Output:

<img width="544" height="107" alt="image" src="https://github.com/user-attachments/assets/638aedeb-f37e-44aa-b14d-76740616f0be" />


### Verify Group Membership

```bash
groups tokyo

groups berlin

groups professor

```
OR (Another way to verify)

```bash
cat /etc/group
```

Output:

<img width="466" height="151" alt="image" src="https://github.com/user-attachments/assets/6f62bfbd-d021-4da3-bacd-efd53306272d" />

OR (Another way to verify)

<img width="389" height="26" alt="image" src="https://github.com/user-attachments/assets/ecd69132-0d36-4b97-984f-868a396d8d01" />

<img width="526" height="72" alt="image" src="https://github.com/user-attachments/assets/ba927527-fe95-4938-92b4-0d89dc79dcd0" />


---


# Task 4: Shared Directory

### Create Shared Directory

```bash
sudo mkdir -p /opt/dev-project
```

Output:

<img width="477" height="60" alt="image" src="https://github.com/user-attachments/assets/4ffeebdd-aa64-4ce6-baf7-2bb596e27de6" />


### Change Group Ownership

```bash
sudo chgrp developers /opt/dev-project
```

Output:

<img width="576" height="85" alt="image" src="https://github.com/user-attachments/assets/6a82a0d2-b067-44aa-ae88-fbc30ff0ae1c" />



### Set Permissions

```bash
sudo chmod 775 /opt/dev-project
```

Output:

<img width="531" height="48" alt="image" src="https://github.com/user-attachments/assets/74e0c7ab-d177-44bb-b033-5c5f1a0f9c1c" />


### Verify Permissions

```bash
ls -ld /opt/dev-project
```

Output:

<img width="531" height="76" alt="image" src="https://github.com/user-attachments/assets/bd22eaf4-b716-4db4-9473-1e30058de33d" />



```bash
drwxrwxr-x
```

### Test File Creation

Create file as tokyo:

```bash
sudo -u tokyo touch /opt/dev-project/tokyo-file.txt
```

Create file as berlin:

```bash
sudo -u berlin touch /opt/dev-project/berlin-file.txt
```

Output:

<img width="614" height="66" alt="image" src="https://github.com/user-attachments/assets/f1faafd2-d057-4e24-8502-0bb5039531cf" />


### Verify Files

```bash
ls -l /opt/dev-project
```

Output:

<img width="445" height="109" alt="image" src="https://github.com/user-attachments/assets/8f5876df-05cd-4044-8ded-b6fb260e3b35" />


---

# Task 5: Team Workspace

### Create User

```bash
sudo useradd -m nairobi
```
Output:

<img width="558" height="66" alt="image" src="https://github.com/user-attachments/assets/cd200fdf-487b-4ca7-a6ab-705be64710d8" />


### Set Password

```bash
sudo passwd nairobi
```

Output:

<img width="362" height="100" alt="image" src="https://github.com/user-attachments/assets/6e718a35-1d71-4932-905a-944d3cdf6c25" />


### Create Group

```bash
sudo groupadd project-team
```

Output:

<img width="411" height="73" alt="image" src="https://github.com/user-attachments/assets/604da22a-2090-4ca1-95cb-3e5806a149a2" />


### Add Users to Group

```bash
sudo usermod -aG project-team nairobi

sudo usermod -aG project-team tokyo
```

Output:

<img width="500" height="104" alt="image" src="https://github.com/user-attachments/assets/53e33367-05d5-4784-b39c-dea15fc266aa" />


### Create Workspace Directory

```bash
sudo mkdir -p /opt/team-workspace
```

Output:

<img width="527" height="57" alt="image" src="https://github.com/user-attachments/assets/72c11579-3975-47fb-a861-ec8b1e366208" />


### Set Group Ownership

```bash
sudo chgrp project-team /opt/team-workspace
```

Output:

<img width="591" height="50" alt="image" src="https://github.com/user-attachments/assets/8db37813-5955-438d-ba0c-315933775af4" />


### Set Permissions

```bash
sudo chmod 775 /opt/team-workspace
```
Output:

<img width="518" height="57" alt="image" src="https://github.com/user-attachments/assets/4cd32016-4baf-475f-a421-956622999a27" />



### Verify Directory Permissions

```bash
ls -ld /opt/team-workspace
```
Output:

<img width="541" height="68" alt="image" src="https://github.com/user-attachments/assets/23107407-c4fd-4da3-b492-651237e8ba91" />


### Test File Creation as Nairobi

```bash
sudo -u nairobi touch /opt/team-workspace/nairobi-file.txt
```

Output:

<img width="671" height="90" alt="image" src="https://github.com/user-attachments/assets/712fd540-2e48-47c8-bbe5-64a091d44ec5" />


### Verify File

```bash
ls -l /opt/team-workspace
```

Output:

<img width="485" height="112" alt="image" src="https://github.com/user-attachments/assets/7bb42e96-846b-4807-acea-539e2764e22d" />


---

# Day 09 Challenge

## Users & Groups Created

### Users
- tokyo
- berlin
- professor
- nairobi

### Groups
- developers
- admins
- project-team

---

# Group Assignments

| User | Groups |
|------|------|
| tokyo | developers, project-team |
| berlin | developers, admins |
| professor | admins |
| nairobi | project-team |

---

# Directories Created

| Directory | Group Owner | Permissions |
|------|------|------|
| /opt/dev-project | developers | 775 |
| /opt/team-workspace | project-team | 775 |

---

# Commands Used

```bash
sudo useradd -m username

sudo passwd username

sudo groupadd groupname

sudo usermod -aG groupname username

groups username

sudo mkdir -p /opt/directory-name

sudo chgrp groupname /opt/directory-name

sudo chmod 775 /opt/directory-name

ls -ld /opt/directory-name

sudo -u username touch /path/file.txt
```

---

# What I Learned

- How Linux users and groups are managed
- How to assign users to multiple groups
- How shared directory permissions work in Linux
- Importance of group ownership and permission settings
- How to test permissions using different users

---

# Troubleshooting

### Permission Denied

Used `sudo` for administrative tasks.

### User Unable to Access Directory

Checked:
- Group membership using:

```bash
groups username
```

- Directory permissions using:

```bash
ls -ld /path
```

- Group ownership using:

```bash
ls -l /opt
```
