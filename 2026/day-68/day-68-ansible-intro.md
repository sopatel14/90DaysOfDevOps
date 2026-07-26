# Day 68: Introduction to Ansible and Inventory Setup

## Ansible Architecture
Ansible operates on an agentless, push-based orchestration architecture. Instead of running continuous background processes (agents) on targeted servers, it executes tasks entirely from a centralized administration machine over standard communication protocols.

```text
               +----------------------------------------+
               |              CONTROL NODE               |
               |        (Local MacBook Control)          |
               +----------------------------------------+
                /         |                  \
     Parses    /          |                   \    Executes
   Inventory  /           |                    \   Modules
             v            v                     v
   +------------+   +------------+        +------------+
   |  Playbook  |   |  Playbook  |        |  Playbook  |
   | (YAML App) |   | (YAML Web) |        |  (YAML DB) |
   +------------+   +------------+        +------------+
         |                |                      |
         | SSH            | SSH                  | SSH
         v                v                      v
+----------------+ +----------------+ +----------------+
|  MANAGED NODE  | |  MANAGED NODE  | |  MANAGED NODE  |
|  (Web Server)  | |  (App Server)  | |  (DB Server)   |
+----------------+ +----------------+ +----------------+
```

### Architectural Components
* **Control Node:** The system where Ansible is physically installed and run (my local MacBook Pro). It parses operational logic and dispatches targeted commands.
* **Managed Nodes:** The remote bare-metal servers or cloud instances managed by the control system (the 3 AWS EC2 instances).
* **Inventory:** A configuration manifest identifying hostnames, connection variables, and infrastructure operational layers.
* **Modules:** Standalone, discrete binaries or script units executed tracking stateful targets (e.g., `apt`, `copy`, `command`).
* **Playbooks:** Reusable, declarative automation blueprints written in structural YAML format mapping out multi-tier machine states.

---

## Lab Environment Setup
* **Infrastructure Provisioning Method:** Deployed programmatically via Terraform.
* **Control Node Engine:** Local macOS environment (MacBook Pro).
* **Managed Node Operating System:** 3 identical remote AWS EC2 instances running **Ubuntu 24.04 LTS** (`t2.micro`).
* **Networking Policy:** Standard AWS Security Groups permitting unrestricted Inbound SSH traffic on **Port 22** strictly isolated to my local management IP.

---

## Inventory Configuration

### `inventory.ini`
```ini
[web]
web-server ansible_host=<PUBLIC_IP_1>

[app]
app-server ansible_host=<PUBLIC_IP_2>

[db]
db-server ansible_host=<PUBLIC_IP_3>

[application:children]
web
app

[all_servers:children]
application
db
```



### `ansible.cfg`
```ini
[defaults]
inventory = inventory.ini
host_key_checking = False
remote_user = ubuntu
private_key_file = ~/ansible-challenge.pem
timeout = 30
stdout_callback = yaml
```

---

## Verification & Connection Check

Running the unified connection ping checks structural SSH path reachability directly via internal Python modules across all targets.

```bash
ansible all -m ping
```

### Execution Proof

<img width="550" height="276" alt="image" src="https://github.com/user-attachments/assets/77a41c64-cf40-467b-a135-1dbb3060498f" />


---

## Ad-Hoc Commands and Outputs

### 1. Check Remote Infrastructure Uptime
```bash
ansible all -m command -a "uptime"
```
**Output Summary:** Returns system running runtime clocks, current logged-in administrative sessions, and average load metrics.

<img width="639" height="161" alt="image" src="https://github.com/user-attachments/assets/ceacad49-02f8-467b-840b-0dd6f89941e3" />


### 2. Verify Available Memory Capacity (Web Group Only)
```bash
ansible web -m command -a "free -h"
```
**Output Summary:** Validates physical and swap memory allocations across targeted web tiers in human-readable notation.

<img width="637" height="123" alt="image" src="https://github.com/user-attachments/assets/eb8b0434-dbb5-4e5d-8e12-81c6fef1ef6b" />


### 3. Check Disk Storage Consumption
```bash
ansible all -m command -a "df -h"
```
**Output Summary:** Lists storage space file system mapping and utilization trends across all mounted storage volumes.

<img width="754" height="820" alt="image" src="https://github.com/user-attachments/assets/9c057779-8e29-4fe6-aa6a-de9a1f359bc8" />


### 4. Install Git Package with Privilege Escalation
```bash
ansible web -m apt -a "name=git state=present update_cache=yes" --become
```
**Output Summary:** Leverages structural OS tools (`apt`) to check package installations and updates caches securely.

<img width="872" height="157" alt="image" src="https://github.com/user-attachments/assets/c7dfad1e-5ac3-45e5-a261-e06cdf938b35" />


### 5. File Transmission & Verification
```bash
echo "Hello from Ansible" > hello.txt
ansible all -m copy -a "src=hello.txt dest=/tmp/hello.txt"
ansible all -m command -a "cat /tmp/hello.txt"
```
**Output Summary:** Transmits a local tracking file outward to all target `/tmp` staging paths, then prints the content text verification.

<img width="850" height="943" alt="image" src="https://github.com/user-attachments/assets/7c4f6448-4660-4089-89da-90d9b13dc6a1" />


### Understanding the `--become` Flag
The `--become` argument triggers operational **privilege escalation** (running commands as `root` via `sudo`). It is strictly mandatory when carrying out tasks that require root permissions, such as managing system services, updating firewall configurations, or installing system packages.

---

## Technical Comparison: Command Module vs. Shell Module

| Capability Feature | `command` Module (Default) | `shell` Module |
| :--- | :--- | :--- |
| **Shell Environment** | Does not launch a shell environment on target. | Spawns a full `/bin/sh` shell session. |
| **Security Integrity** | High. Safe from shell injection vulnerabilities. | Lower. Vulnerable to shell interpolation exploits. |
| **Pipe / Redirect Support** | Unsupported. Treats `<`, `>`, and `\|` as raw text arguments. | Fully supported. Can process pipelines and standard output routing. |
| **Environment Variables** | Cannot read host shell variables like `$HOME` or `$PATH`. | Can interpret and resolve all host system environment variables. |
