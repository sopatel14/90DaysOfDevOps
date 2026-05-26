# Day 11 – File Ownership Challenge (chown & chgrp)

# Task 1: Understanding Ownership

## Check File Ownership

```bash
ls -l
```

Output:

<img width="498" height="213" alt="image" src="https://github.com/user-attachments/assets/29ebdf4e-f917-4359-8498-fa973b895a2a" />


---

## Ownership Format

```text
-rw-r--r-- 1 owner group size date filename
```

### Explanation

| Field | Meaning |
|------|------|
| owner | User who owns the file |
| group | Group associated with the file |

---

## Difference Between Owner and Group

- **Owner** → The main user who controls the file
- **Group** → Multiple users in the same group can access the file based on group permissions

---

# Task 2: Basic chown Operations

## Create File

```bash
touch devops-file.txt
```

---

## Check Current Owner

```bash
ls -l devops-file.txt
```

Output:

<img width="461" height="99" alt="image" src="https://github.com/user-attachments/assets/7e2345eb-5144-46ec-a602-bffd68c887b2" />


---

## Change Owner to tokyo

```bash
sudo chown tokyo devops-file.txt
```

Verify:

```bash
ls -l devops-file.txt
```

Output:

<img width="491" height="103" alt="image" src="https://github.com/user-attachments/assets/bd2230e3-789b-4116-8d3e-a4d7cb260e5e" />


---

## Change Owner to berlin

```bash
sudo chown berlin devops-file.txt
```

Verify:

```bash
ls -l devops-file.txt
```

Output:

<img width="507" height="84" alt="image" src="https://github.com/user-attachments/assets/89e2138c-b980-43ac-9643-259264f80e74" />


---

# Task 3: Basic chgrp Operations

## Create File

```bash
touch team-notes.txt
```

---

## Check Current Group

```bash
ls -l team-notes.txt
```

Output:

<img width="436" height="84" alt="image" src="https://github.com/user-attachments/assets/61f73e7e-3173-475f-9fea-c931e8aaaefc" />


---

## Create Group

```bash
sudo groupadd heist-team
```
Output:

<img width="474" height="58" alt="image" src="https://github.com/user-attachments/assets/cf263d3b-bed4-4f9c-a6e7-b96cfb4730d0" />


---

## Change File Group

```bash
sudo chgrp heist-team team-notes.txt
```

Verify:

```bash
ls -l team-notes.txt
```

Output:

<img width="536" height="95" alt="image" src="https://github.com/user-attachments/assets/540e1eb1-bb31-4e32-8fd9-933ca1dc0cf5" />


---

# Task 4: Combined Owner & Group Change

## Create File

```bash
touch project-config.yaml
```
Output:

<img width="488" height="73" alt="image" src="https://github.com/user-attachments/assets/5f7dc9e0-f8bf-442e-a4fa-98e4461661a2" />


---

## Change Owner and Group Together

```bash
sudo chown professor:heist-team project-config.yaml
```

Verify:

```bash
ls -l project-config.yaml
```
Output:

<img width="523" height="65" alt="image" src="https://github.com/user-attachments/assets/cfb26257-d5c5-4b82-b273-379ffc310c91" />


---

## Create Directory

```bash
mkdir app-logs
```

---

## Change Owner and Group for Directory

```bash
sudo chown berlin:heist-team app-logs
```

Output:

<img width="553" height="40" alt="image" src="https://github.com/user-attachments/assets/02e48a82-a4e1-43f7-adfd-9b8cbde4163a" />


Verify:

```bash
ls -ld app-logs
```
Output:

<img width="493" height="62" alt="image" src="https://github.com/user-attachments/assets/230d427a-4e4e-42dd-bdec-c57524dbd753" />


---

# Task 5: Recursive Ownership

## Create Directory Structure

```bash
mkdir -p heist-project/vault

  mkdir -p heist-project/plans

touch heist-project/vault/gold.txt

touch heist-project/plans/strategy.conf
```

Output:

<img width="568" height="108" alt="image" src="https://github.com/user-attachments/assets/2b399e4c-2ad5-43db-8080-f8edc8f5c489" />


---

## Create Group

```bash
sudo groupadd planners
```

---

## Change Ownership Recursively

```bash
sudo chown -R professor:planners heist-project/
```

---

## Verify Recursive Changes

```bash
ls -lR heist-project/
```

Output:

<img width="559" height="307" alt="image" src="https://github.com/user-attachments/assets/1cda44bc-f044-4402-925a-0bf338881079" />


---

# Task 6: Practice Challenge

## Create Groups

```bash
sudo groupadd vault-team

sudo groupadd tech-team
```

---

## Create Directory

```bash
mkdir bank-heist
```

---

## Create Files

```bash
touch bank-heist/access-codes.txt

touch bank-heist/blueprints.pdf

touch bank-heist/escape-plan.txt
```

Output:

<img width="491" height="127" alt="image" src="https://github.com/user-attachments/assets/46c9f566-ccef-4520-ae87-ba36a59c7669" />


---

## Set File Ownership

### access-codes.txt

```bash
sudo chown tokyo:vault-team bank-heist/access-codes.txt
```

---

### blueprints.pdf

```bash
sudo chown berlin:tech-team bank-heist/blueprints.pdf
```

---

### escape-plan.txt

```bash
sudo chown nairobi:vault-team bank-heist/escape-plan.txt
```

---

## Verify Ownership

```bash
ls -l bank-heist/
```

Output:

<img width="496" height="106" alt="image" src="https://github.com/user-attachments/assets/ad703447-58bc-4730-93ab-3bc9b237dee3" />


---

# Key Commands Reference

## View Ownership

```bash
ls -l filename
```

---

## Change Owner Only

```bash
sudo chown newowner filename
```

---

## Change Group Only

```bash
sudo chgrp newgroup filename
```

---

## Change Owner and Group Together

```bash
sudo chown owner:group filename
```

---

## Recursive Ownership Change

```bash
sudo chown -R owner:group directory/
```

---

## Change Only Group Using chown

```bash
sudo chown :groupname filename
```

---


# Day 11 Challenge

## Files & Directories Created

### Files
- devops-file.txt
- team-notes.txt
- project-config.yaml
- access-codes.txt
- blueprints.pdf
- escape-plan.txt
- gold.txt
- strategy.conf

### Directories
- app-logs/
- heist-project/
- heist-project/vault/
- heist-project/plans/
- bank-heist/

---

# Ownership Changes

| File/Directory | Before | After |
|------|------|------|
| devops-file.txt | ubuntu:ubuntu | tokyo:ubuntu |
| devops-file.txt | tokyo:ubuntu | berlin:ubuntu |
| team-notes.txt | ubuntu:ubuntu | ubuntu:heist-team |
| project-config.yaml | ubuntu:ubuntu | professor:heist-team |
| app-logs/ | ubuntu:ubuntu | berlin:heist-team |
| heist-project/ | ubuntu:ubuntu | professor:planners |
| access-codes.txt | ubuntu:ubuntu | tokyo:vault-team |
| blueprints.pdf | ubuntu:ubuntu | berlin:tech-team |
| escape-plan.txt | ubuntu:ubuntu | nairobi:vault-team |

---

# Commands Used

```bash
ls -l

touch filename

sudo chown owner filename

sudo chgrp group filename

sudo chown owner:group filename

sudo chown -R owner:group directory/

mkdir -p directory-name

sudo groupadd groupname
```

---

# What I Learned

- Difference between file owner and file group
- How to change ownership using `chown`
- How to change groups using `chgrp`
- Importance of recursive ownership changes for directories
- How Linux permissions and ownership work together for secure access control

---

# Why This Matters for DevOps

Proper file ownership is important in real DevOps environments for:

- Application deployments
- Shared team access
- CI/CD pipeline artifacts
- Docker container permissions
- Log file management
- Secure production environments
