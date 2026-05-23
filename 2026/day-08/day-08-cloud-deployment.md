# Part 1: Launch Cloud Instance & SSH Access

## Step 1: Launch EC2 Instance

Launched an Ubuntu EC2 instance on AWS Free Tier.

---

## Step 2: Connect via SSH

```bash id="40h7k7"
ssh -i my-key.pem ubuntu@<your-public-ip>
```

SSH connection

<img width="1316" height="532" alt="image" src="https://github.com/user-attachments/assets/de14fcf7-c6ee-4671-9a2e-bcbd0f9020e0" />

<img width="966" height="558" alt="image" src="https://github.com/user-attachments/assets/d46e8e41-96e4-4ac2-8353-7a4fc0a0b8a5" />

Successfully connected to the remote Linux server.

# Part 2: Install Docker & Nginx

## Step 1: Update System Packages

```bash
sudo apt update && sudo apt upgrade -y
```

Output:

<img width="630" height="426" alt="image" src="https://github.com/user-attachments/assets/f39135b5-5893-46d3-8215-daa9e3897d52" />


## Step 2: Install Docker

```bash
sudo apt install docker.io -y
```

Output:

<img width="456" height="90" alt="image" src="https://github.com/user-attachments/assets/1712d47c-23fa-4bb0-81d4-00a4f593740d" />


Verify Docker installation:

```bash
docker --version
```

Output :

<img width="352" height="61" alt="image" src="https://github.com/user-attachments/assets/83d0e9f7-8853-4529-b962-c4f2cf1af769" />

Start and enable Docker:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

Output :

<img width="416" height="75" alt="image" src="https://github.com/user-attachments/assets/5178defe-f44c-4b56-993f-f01bcf46911e" />


Check Docker status:

```bash
sudo systemctl status docker
```

Output:

<img width="666" height="438" alt="image" src="https://github.com/user-attachments/assets/3dc3f25c-83d0-4117-9a70-8e0f3e9f2f54" />

## Step 3: Install Nginx

```bash
sudo apt install nginx -y
```
Output :

<img width="459" height="102" alt="image" src="https://github.com/user-attachments/assets/0ca60bcb-ebd5-4f42-99bd-703a44efc325" />



Verify Nginx status:

```bash
sudo systemctl status nginx
```
Output :

<img width="669" height="350" alt="image" src="https://github.com/user-attachments/assets/61985128-d260-4982-af54-0eff9b4e552a" />


Enable Nginx on boot:

```bash
sudo systemctl enable nginx
```

Output:

<img width="664" height="129" alt="image" src="https://github.com/user-attachments/assets/57ecd68a-dabb-4d1a-b5e7-abd9376887b4" />

# Part 3: Security Group Configuration

Configured inbound rules in the AWS Security Group:

Steps :

1.Go to Security in your AWS EC2 Instance

<img width="949" height="320" alt="image" src="https://github.com/user-attachments/assets/6012e2ca-3e40-40c0-ab8b-e30b427b6861" />

2. Click on Security Groups > Inbound rules

<img width="961" height="179" alt="image" src="https://github.com/user-attachments/assets/a9933a14-68e3-42f1-83f7-0959435d2e54" />

3. Click on Edit Inbound Rules
   
<img width="961" height="179" alt="image" src="https://github.com/user-attachments/assets/0b062b63-1f65-4eb0-8e67-85c8ed45f765" />

4. Click on Add rule and select HTTP

<img width="1644" height="399" alt="image" src="https://github.com/user-attachments/assets/65b94dbe-5848-45e5-a83e-44e2a87b1726" />

## Test Web Access

Opened browser and visited:

```bash
http://<your-public-ip>
```
Output:

<img width="1188" height="724" alt="image" src="https://github.com/user-attachments/assets/397b7bd0-488e-40ed-9042-59c3833115f4" />

Successfully viewed the default Nginx welcome page.
  
  
# Part 4: Extract Nginx Logs

## Step 1: View Nginx Logs

```bash
sudo tail -n 20 /var/log/nginx/access.log
```

Output:

<img width="670" height="226" alt="image" src="https://github.com/user-attachments/assets/1d4a880c-d799-4289-84ca-0862e955b636" />


## Nginx Error Log Check

```bash
sudo tail -n 20 /var/log/nginx/error.log
```

Output:

<img width="589" height="87" alt="image" src="https://github.com/user-attachments/assets/aafa1b86-f517-4c76-bde4-dfc39b3fae68" />

Observation:
No errors were present in the Nginx error log, which indicates the web server was running successfully without issues.


## Step 2: Save Logs to File

```bash
sudo cp /var/log/nginx/access.log ~/nginx-logs.txt
```

Verify file contents:

```bash
cat ~/nginx-logs.txt
```
Output :

<img width="671" height="301" alt="image" src="https://github.com/user-attachments/assets/65461823-0e81-400a-9f05-6d776f5dae51" />



---

## Step 3: Download Log File to Local Machine

### AWS

```bash
scp -i my-key.pem ubuntu@<your-public-ip>:~/nginx-logs.txt .
```

Faced Permission denied error

<img width="659" height="82" alt="image" src="https://github.com/user-attachments/assets/91b15145-f5d1-4b6e-afac-39a360fc0f0e" />

 Ubuntu user does not have permission to read it.

```bash
sudo chown ubuntu:ubuntu ~/nginx-logs.txt
```
<img width="537" height="54" alt="image" src="https://github.com/user-attachments/assets/087502fd-3ccb-4e7c-a0c8-0c3b503bb50e" />

The file is downloaded

<img width="666" height="103" alt="image" src="https://github.com/user-attachments/assets/33b1784f-671e-4df2-8cb5-423c7501cd5b" />

 # Challenges Faced

- Initially forgot to allow HTTP (Port 80) in the Security Group, so the Nginx webpage was not accessible from the browser.

- Faced SSH permission issues with the `.pem` key file and resolved it using:

```bash
chmod 400 my-key.pem
```

- While downloading `nginx-logs.txt` using `scp`, received a `Permission denied` error because the file was created using `sudo` and owned by the `root` user.

Resolved it by changing file ownership:

```bash
sudo chown ubuntu:ubuntu ~/nginx-logs.txt
```

- Learned how to troubleshoot service and permission issues using `systemctl`, `ls -l`, and Linux file ownership commands.

---

# What I Learned

- How to launch and connect to a cloud server using SSH
- Installing and managing services using `systemctl`
- How Nginx serves web pages over HTTP
- Importance of Security Groups and firewall rules
- How to view, copy, and transfer logs securely using `scp`

---

# Why This Matters for DevOps

This exercise teaches important real-world DevOps skills:

- Cloud infrastructure provisioning
- Remote Linux server management
- Web server deployment and configuration
- Security and access control
- Log management and troubleshooting

These are foundational skills used daily in production environments by DevOps engineers.




