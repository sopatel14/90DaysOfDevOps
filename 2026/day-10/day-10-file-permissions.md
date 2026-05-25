# Day 10 – File Permissions & File Operations Challenge

# Task 1: Create Files

## Create Empty File

```bash
touch devops.txt
```

Output:

<img width="515" height="45" alt="image" src="https://github.com/user-attachments/assets/a2db2d79-8615-4a31-8ed2-40d78178dff4" />


---

## Create notes.txt with Content

```bash
echo "Linux file permissions practice" > notes.txt
```

Output:

<img width="546" height="76" alt="image" src="https://github.com/user-attachments/assets/34dbac3c-6635-4106-b967-b557e3be165a" />


---

## Create script.sh using vim

```bash
vim script.sh
```

Add the following content:

```bash
echo "Hello DevOps"
```

Save and exit:
- Press `Esc`
- Type `:wq`
- Press `Enter`

Output:

<img width="299" height="385" alt="image" src="https://github.com/user-attachments/assets/a880c0f7-e964-43f3-9360-c9b8621c0005" />



---

## Verify Files and Permissions

```bash
ls -l
```

Output:

<img width="477" height="189" alt="image" src="https://github.com/user-attachments/assets/a389d802-689c-4a7d-be7b-38282d5156c5" />


---

# Task 2: Read Files

## Read notes.txt

```bash
cat notes.txt
```
Output:

<img width="380" height="64" alt="image" src="https://github.com/user-attachments/assets/a229fb0c-af73-4979-a9b0-5da43d257ca6" />

---

## Open script.sh in Read-Only Mode

```bash
vim -R script.sh
```

Output:

<img width="333" height="429" alt="image" src="https://github.com/user-attachments/assets/7a7cb270-f850-4137-bb50-d15aac9707f2" />


---

## Display First 5 Lines of /etc/passwd

```bash
head -n 5 /etc/passwd
```
Output:

<img width="379" height="137" alt="image" src="https://github.com/user-attachments/assets/07785147-2133-441a-934a-45e3091bac2a" />


---

## Display Last 5 Lines of /etc/passwd

```bash
tail -n 5 /etc/passwd
```
<img width="444" height="133" alt="image" src="https://github.com/user-attachments/assets/4d775fc0-bcc2-467f-a0df-cd076c5361a4" />

---

# Task 3: Understand Permissions

## Check File Permissions

```bash
ls -l devops.txt notes.txt script.sh
```

Example:

```bash
-rw-rw-r-- 1 ubuntu ubuntu 0 devops.txt
-rw-rw-r-- 1 ubuntu ubuntu 35 notes.txt
-rw-rw-r-- 1 ubuntu ubuntu 21 script.sh
```

Output:

<img width="467" height="101" alt="image" src="https://github.com/user-attachments/assets/431ae114-3225-4ad4-8153-e9b777adec66" />


---

## Permission Format

```text
rwxrwxrwx
│ │ │
│ │ └── Others
│ └──── Group
└────── Owner
```

### Permission Values

| Permission | Value |
|------|------|
| r (read) | 4 |
| w (write) | 2 |
| x (execute) | 1 |

---

## Current Permissions Explanation

| File | Owner | Group | Others |
|------|------|------|------|
| devops.txt | read, write | read, write | read |
| notes.txt | read, write | read, write | read |
| script.sh | read, write | read, write | read |

Currently, no file has execute permission.

---

# Task 4: Modify Permissions

## Make script.sh Executable

```bash
chmod +x script.sh
```

Verify:

```bash
ls -l script.sh
```

Output:

<img width="493" height="81" alt="image" src="https://github.com/user-attachments/assets/a59f68de-90f6-49ec-b4f9-23865c41af08" />


Run the script:

```bash
./script.sh
```

Output:

<img width="294" height="58" alt="image" src="https://github.com/user-attachments/assets/4d2187cd-cbe5-426e-8cc1-d39cecdc36cd" />


---

## Make devops.txt Read-Only

Remove write permission for all users:

```bash
chmod 444 devops.txt
```

Verify:

```bash
ls -l devops.txt
```

Output:

<img width="460" height="118" alt="image" src="https://github.com/user-attachments/assets/0caefef6-c213-459f-8535-a1d211b952b6" />


---

## Set notes.txt to 640

```bash
chmod 640 notes.txt
```

Verify:

```bash
ls -l notes.txt
```

Output:

<img width="460" height="100" alt="image" src="https://github.com/user-attachments/assets/33f5e916-a47d-4f54-b743-82ce37e1933a" />


Explanation:
- Owner → read, write
- Group → read only
- Others → no permission

---

## Create project Directory with 755 Permissions

```bash
mkdir project
```

Set permissions:

```bash
c```

Verify:

```bash
ls -ld project
```

Output:

<img width="457" height="119" alt="image" src="https://github.com/user-attachments/assets/5cbab5b1-f5dc-4a45-8560-4c425529b002" />


---

# Task 5: Test Permissions

## Try Writing to Read-Only File

```bash
echo "Testing" >> devops.txt
```

Error:

<img width="481" height="86" alt="image" src="https://github.com/user-attachments/assets/6b071722-7b64-481a-8648-c2af7c811767" />


---

## Remove Execute Permission from script.sh

```bash
chmod -x script.sh
```

Try executing:

```bash
./script.sh
```

Error:



---

# Day 10 Challenge

## Files Created

- devops.txt
- notes.txt
- script.sh
- project/

---

# Permission Changes

| File/Directory | Before | After |
|------|------|------|
| script.sh | -rw-rw-r-- | -rwxrwxr-x |
| devops.txt | -rw-rw-r-- | -r--r--r-- |
| notes.txt | -rw-rw-r-- | -rw-r----- |
| project/ | default | drwxr-xr-x |

---

# Commands Used

```bash
touch devops.txt

echo "Linux file permissions practice" > notes.txt

vim script.sh

cat notes.txt

vim -R script.sh

head -n 5 /etc/passwd

tail -n 5 /etc/passwd

ls -l

chmod +x script.sh

./script.sh

chmod a-w devops.txt

chmod 640 notes.txt

mkdir project

chmod 755 project

ls -ld project
```

---

# What I Learned

- How Linux file permissions work using read, write, and execute
- How to modify permissions using symbolic and numeric chmod methods
- Importance of execute permissions for scripts
- Difference between file and directory permissions
- How permission errors help secure Linux systems
