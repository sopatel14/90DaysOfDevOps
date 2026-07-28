# Day 70 — Variables, Facts, Conditionals & Loops in Ansible

## Task 1: Variables in Playbooks

### Theory
Variables make playbooks dynamic and reusable instead of hardcoded. They can be defined at several levels:

- **Playbook level** — `vars:` block
- **Inventory level** — `group_vars/` and `host_vars/`
- **Command line** — `-e` flag
- **Task level** — `set_fact`

**Variable precedence (lowest → highest):**

1. Role defaults
2. Inventory `group_vars/all`
3. Inventory `group_vars/<group_name>`
4. Inventory `host_vars/<host_name>`
5. Playbook `vars`
6. Task vars
7. Extra vars (`-e`)

### Implementation — `variables-demo.yml`

```yaml
---
- name: Variables demo
  hosts: all
  vars:
    app_name: terraweek-app
    app_port: 8080
    app_dir: "/opt/{{ app_name }}"
    packages:
      - git
      - curl
      - wget

  tasks:
    - name: Show deployment target
      debug:
        msg: "Deploying {{ app_name }} on port {{ app_port }} to {{ app_dir }}"
```

### Testing Variable Override

**Run 1 — playbook defaults:**

```bash
ansible-playbook playbooks/variables-demo.yml
```

```text
TASK [Show deployment target] ***************************
ok: [web-server] => {
    "msg": "Deploying terraweek-app on port 8080 to /opt/terraweek-app"
}
```

📸 **Verification Screenshot:** 

<img width="2068" height="1170" alt="image" src="https://github.com/user-attachments/assets/e22493b8-fe91-4d2e-bab3-417456c26a1d" />


**Run 2 — CLI override with `-e`:**

```bash
ansible-playbook playbooks/variables-demo.yml -e "app_name=my-custom-app app_port=9090"
```

```text
TASK [Show deployment target] ***************************
ok: [web-server] => {
    "msg": "Deploying my-custom-app on port 9090 to /opt/my-custom-app"
}
```

📸 **Verification Screenshot:** 

<img width="2056" height="1096" alt="image" src="https://github.com/user-attachments/assets/7b7870c3-bc15-43ca-9cde-3611633cc70f" />


`app_dir` recalculates automatically because it references `{{ app_name }}` — confirming extra vars (`-e`) sit at the top of the precedence chain.

---

## Task 2: `group_vars` and `host_vars`

Variables shouldn't live inside playbooks — they belong in dedicated inventory directories so the same playbook can drive different environments.

### 1. Directory Structure

```text
ansible-practice/
  inventory.ini
  ansible.cfg
  group_vars/
    all.yml
    web.yml
    db.yml
  host_vars/
    web-server.yml
  playbooks/
    site.yml
```


### 2. Variable Configuration Files

**`group_vars/all.yml`** — applies to every host:

```yaml
---
ntp_server: pool.ntp.org
app_env: development
common_packages:
  - vim
  - htop
  - tree
```

**`group_vars/web.yml`** — applies only to the `[web]` group:

```yaml
---
http_port: 80
max_connections: 1000
web_packages:
  - nginx
```

**`group_vars/db.yml`** — applies only to the `[db]` group:

```yaml
---
db_port: 3306
db_packages:
  - mysql-server
```

**`host_vars/web-server.yml`** — applies only to the host `web-server`:

```yaml
---
max_connections: 2000
custom_message: "This is the primary web server"
```

### 3. Playbook — `playbooks/site.yml`

```yaml
---
- name: Apply common config
  hosts: all
  become: true
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Install common packages
      apt:
        name: "{{ common_packages }}"
        state: present

    - name: Show environment
      debug:
        msg: "Environment: {{ app_env }}"

- name: Configure web servers
  hosts: web
  become: true
  tasks:
    - name: Show web config
      debug:
        msg: "HTTP port: {{ http_port }}, Max connections: {{ max_connections }}"

    - name: Show host-specific message
      debug:
        msg: "{{ custom_message }}"
```

### 4. Execution & Observation

**Run 1 — default behavior:**

```bash
ansible-playbook playbooks/site.yml
```

```text
TASK [Show web config] ***********************************
ok: [web-server] => {
    "msg": "HTTP port: 80, Max connections: 2000"
}
TASK [Show host-specific message] ************************
ok: [web-server] => {
    "msg": "This is the primary web server"
}
```

📸 **Verification Screenshot:** 

<img width="2064" height="1430" alt="image" src="https://github.com/user-attachments/assets/7ca4d01d-21ad-4e4b-b28b-53c95c52354b" />


`web-server` reports `2000` (from `host_vars`) instead of the group default of `1000` — confirming `host_vars` outranks `group_vars`.

**Run 2 — CLI override:**

```bash
ansible-playbook playbooks/site.yml -e "max_connections=3000 app_env=production"
```

```text
TASK [Show environment] **********************************
ok: [web-server] => { "msg": "Environment: production" }
TASK [Show web config] ***********************************
ok: [web-server] => { "msg": "HTTP port: 80, Max connections: 3000" }
```

📸 **Verification Screenshot:** 

<img width="2060" height="1410" alt="image" src="https://github.com/user-attachments/assets/dc223e06-65a2-4db5-95a0-6a9b01728d24" />


All hosts now report `3000` and `production`, since extra vars override everything — including `host_vars`.

### 5. Variable Precedence Summary

| Priority | Source |
|---|---|
| Lowest | `group_vars/all.yml` |
| ↑ | `group_vars/<group_name>.yml` |
| ↑ | `host_vars/<host_name>.yml` |
| ↑ | Playbook `vars` |
| Highest | Extra vars `-e` (CLI) |

---

## Task 3: Ansible Facts — Gathering System Information

Ansible automatically collects **facts** about each managed node — OS, IP addresses, memory, CPU, disks, and hundreds of other metrics.

### 1. Discovering Facts via CLI

```bash
# View everything
ansible web-server -m setup

# Broad OS family
ansible web-server -m setup -a "filter=ansible_os_family"

# Detailed distribution info
ansible web-server -m setup -a "filter=ansible_distribution*"

# Total system memory (MB)
ansible web-server -m setup -a "filter=ansible_memtotal_mb"

# Default network gateway
ansible web-server -m setup -a "filter=ansible_default_ipv4"
```

```text
web-server | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "Ubuntu",
        "ansible_distribution_version": "22.04",
        "ansible_memtotal_mb": 957
    }
}
```

📸 **Verification Screenshot:** 

<img width="1680" height="1522" alt="image" src="https://github.com/user-attachments/assets/dda5bdf3-6399-4faa-8fe2-b2be3f121fa1" />

### 2. Using Facts in a Playbook — `playbooks/facts-demo.yml`

```yaml
---
- name: Facts demo
  hosts: all
  gather_facts: yes
  tasks:
    - name: Show OS info
      debug:
        msg: >
          Hostname: {{ ansible_hostname }},
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }},
          RAM: {{ ansible_memtotal_mb }}MB,
          IP: {{ ansible_default_ipv4.address }}

    - name: Show all network interfaces
      debug:
        var: ansible_interfaces
```

```bash
ansible-playbook playbooks/facts-demo.yml
```

```text
TASK [Show OS info] ***************************************
ok: [web-server] => {
    "msg": "Hostname: web-server, OS: Ubuntu 22.04, RAM: 957MB, IP: 172.31.14.22"
}
```

📸 **Verification Screenshot:** 

<img width="2080" height="1408" alt="image" src="https://github.com/user-attachments/assets/f48514a7-9a10-4373-b167-17562baea62c" />


### 3. Five Critical Facts Used in Production Playbooks

| Fact | Why it matters |
|---|---|
| `ansible_distribution` / `ansible_os_family` | Drives `when:` conditionals so the right package manager (`apt` vs `yum`/`dnf`) runs on the right OS in cross-platform playbooks. |
| `ansible_default_ipv4.address` | Injected into app/Nginx/database configs so services bind to the host's real IP instead of a hardcoded value. |
| `ansible_memtotal_mb` | Sizes resource allocations dynamically — e.g. setting a database buffer pool to a percentage of total RAM. |
| `ansible_virtualization_role` / `ansible_virtualization_type` | Skips bare-metal-only steps (hardware logging, disk monitoring) when running inside containers or cloud VMs. |
| `ansible_processor_vcpus` | Maps worker/thread counts (e.g. Nginx worker processes) directly to available CPU cores. |

---

## Task 4: Conditionals with `when`

Tasks shouldn't always run on every host — `when` gates execution on host groups, gathered facts, or defined variables.

### Playbook — `playbooks/conditional-demo.yml`

```yaml
---
- name: Conditional tasks demo
  hosts: all
  become: true

  tasks:
    - name: Update apt cache (only on web or db servers)
      apt:
        update_cache: yes
        cache_valid_time: 3600
      when: "'web' in group_names or 'db' in group_names"

    - name: Install Nginx (only on web servers)
      apt:
        name: nginx
        state: present
      when: "'web' in group_names"

    - name: Install MySQL (only on db servers)
      apt:
        name: mysql-server
        state: present
      when: "'db' in group_names"

    - name: Show warning on low memory hosts
      debug:
        msg: "WARNING: This host has less than 1GB RAM"
      when: ansible_memtotal_mb < 1024

    - name: Run only on Amazon Linux
      debug:
        msg: "This is an Amazon Linux machine"
      when: ansible_distribution == "Amazon"

    - name: Run only on Ubuntu
      debug:
        msg: "This is an Ubuntu machine"
      when: ansible_distribution == "Ubuntu"

    - name: Run only in production
      debug:
        msg: "Production settings applied"
      when: app_env is defined and app_env == "production"

    - name: Multiple conditions (AND)
      debug:
        msg: "Web server with enough memory"
      when:
        - "'web' in group_names"
        - ansible_memtotal_mb >= 512

    - name: OR condition
      debug:
        msg: "Either web or app server"
      when: "'web' in group_names or 'app' in group_names"
```

```bash
ansible-playbook playbooks/conditional-demo.yml
```

```text
TASK [Install Nginx (only on web servers)] ***************
changed: [web-server]
skipping: [db-server]
skipping: [app-server]

TASK [Install MySQL (only on db servers)] *****************
skipping: [web-server]
changed: [db-server]
skipping: [app-server]

TASK [Run only on Amazon Linux] ****************************
skipping: [web-server]
skipping: [db-server]
skipping: [app-server]

TASK [Run only on Ubuntu] ***********************************
ok: [web-server] => { "msg": "This is an Ubuntu machine" }
ok: [db-server]  => { "msg": "This is an Ubuntu machine" }
ok: [app-server] => { "msg": "This is an Ubuntu machine" }
```

📸 **Verification Screenshot:** 

<img width="2022" height="2062" alt="image" src="https://github.com/user-attachments/assets/81e3e3db-2f99-4a7d-bdf2-1b97beec8ec9" />

**Are tasks correctly skipping on mismatched hosts? Yes.**

- **Group filtering** — `Install Nginx` runs on `web-server` but reports `skipping` on `db-server`/`app-server`.
- **OS filtering** — `Run only on Ubuntu` succeeds on every host; `Run only on Amazon Linux` cleanly skips everywhere, since `ansible_distribution` resolved to `"Ubuntu"` from gathered facts.
- **Variable presence** — the production step skips unless explicitly enabled with `-e "app_env=production"`.

---

## Task 5: Ansible Loops — Iterating Over Items

### Playbook — `playbooks/loops-demo.yml`

```yaml
---
- name: Loops demo
  hosts: all
  become: true

  vars:
    users:
      - name: deploy
        groups: sudo
      - name: monitor
        groups: sudo
      - name: appuser
        groups: users

    directories:
      - /opt/app/logs
      - /opt/app/config
      - /opt/app/data
      - /opt/app/tmp

  tasks:
    - name: Create multiple users
      user:
        name: "{{ item.name }}"
        groups: "{{ item.groups }}"
        state: present
      loop: "{{ users }}"

    - name: Create multiple directories
      file:
        path: "{{ item }}"
        state: directory
        mode: '0755'
      loop: "{{ directories }}"

    - name: Install multiple packages
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - git
        - curl
        - unzip
        - jq

    - name: Print each user created
      debug:
        msg: "Created user {{ item.name }} in group {{ item.groups }}"
      loop: "{{ users }}"
```

```bash
ansible-playbook playbooks/loops-demo.yml
```

```text
TASK [Create multiple users] *******************************
changed: [web-server] => (item={'name': 'deploy', 'groups': 'sudo'})
changed: [web-server] => (item={'name': 'monitor', 'groups': 'sudo'})
changed: [web-server] => (item={'name': 'appuser', 'groups': 'users'})

TASK [Print each user created] *****************************
ok: [web-server] => (item={'name': 'deploy', 'groups': 'sudo'}) => {
    "msg": "Created user deploy in group sudo"
}
```

📸 **Verification Screenshot:** 

<img width="1856" height="1366" alt="image" src="https://github.com/user-attachments/assets/002ddd03-5cb8-4118-a634-0d48001ade76" />


> On resource-constrained instances (e.g. 1GB RAM micro instances), heavy loops can trigger out-of-memory errors during package installs. Mitigate with added swap space, or throttle concurrency with `--forks 1`.

### `loop` vs `with_items`

| Aspect | `loop` | `with_items` |
|---|---|---|
| Status | Modern, standardized (Ansible 2.5+); the recommended default | Legacy `with_<lookup>` syntax, kept for backward compatibility |
| Nested lists | Passed through as-is — needs `\| flatten` to flatten manually | Implicitly auto-flattens nested lists |
| Architecture | Native loop inside Ansible's core engine | Goes through the Jinja2 lookup-plugin framework, adding a small parsing overhead |

**Takeaway:** default to `loop` for new playbooks; `with_items` still works but is being phased out.

---

## Task 6: Register, Debug, and Combine Everything — `server-report.yml`

This playbook ties together variables, facts, `register`, conditionals, and dynamic file generation into one real-world reporting script.

### Playbook — `playbooks/server-report.yml`

```yaml
---
- name: Server Health Report
  hosts: all

  tasks:
    - name: Check disk space
      command: df -h /
      register: disk_result

    - name: Check memory
      command: free -m
      register: memory_result

    - name: Check running services
      shell: systemctl list-units --type=service --state=running | head -20
      register: services_result

    - name: Generate report
      debug:
        msg:
          - "========== {{ inventory_hostname }} =========="
          - "OS: {{ ansible_distribution }} {{ ansible_distribution_version }}"
          - "IP: {{ ansible_default_ipv4.address }}"
          - "RAM: {{ ansible_memtotal_mb }}MB"
          - "Disk: {{ disk_result.stdout_lines[1] }}"
          - "Running services (first 20): {{ services_result.stdout_lines | length }}"

    - name: Flag if disk is critically low
      debug:
        msg: "ALERT: Check disk space on {{ inventory_hostname }}"
      when: "'9[0-9]%' in disk_result.stdout or '100%' in disk_result.stdout"

    - name: Save report to file
      copy:
        content: |
          Server: {{ inventory_hostname }}
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          IP: {{ ansible_default_ipv4.address }}
          RAM: {{ ansible_memtotal_mb }}MB
          Disk: {{ disk_result.stdout }}
          Checked at: {{ ansible_date_time.iso8601 }}
        dest: "/tmp/server-report-{{ inventory_hostname }}.txt"
      become: true
```

### Execution

```bash
ansible-playbook playbooks/server-report.yml
```

```text
TASK [Generate report] ***************************************
ok: [web-server] => {
    "msg": [
        "========== web-server ==========",
        "OS: Ubuntu 22.04",
        "IP: 172.31.14.22",
        "RAM: 957MB",
        "Disk: /dev/xvda1  8.0G  3.2G  4.4G  43% /",
        "Running services (first 20): 20"
    ]
}
```

📸 **Verification Screenshot:** 

<img width="1930" height="2066" alt="image" src="https://github.com/user-attachments/assets/93a434dd-a2bc-4515-b0fb-1d13cbcca3aa" />



### Remote Verification

```bash
ansible web-server -m shell -a "cat /tmp/server-report-web-server.txt"
```

```text
web-server | CHANGED | rc=0 >>
Server: web-server
OS: Ubuntu 22.04
IP: 172.31.14.22
RAM: 957MB
Disk: /dev/xvda1  8.0G  3.2G  4.4G  43% /
Checked at: 2026-07-28T09:14:52Z
```

📸 **Verification Screenshot:** 

<img width="1772" height="324" alt="image" src="https://github.com/user-attachments/assets/7f7d1f75-8c32-4904-9576-854c84aa63e9" />



### Verification Analysis

**Is `/tmp/server-report-*.txt` accurate? Yes.**

- **`register`** captured the raw `df -h /` output into `disk_result.stdout` and injected it verbatim into the report template.
- **Facts** (`ansible_distribution`, `ansible_default_ipv4.address`, `ansible_memtotal_mb`) matched the target machine's real configuration, since they're gathered live at runtime — not hardcoded.
- **`ansible_date_time.iso8601`** stamped the exact UTC execution time, confirming the report reflects a live run rather than cached or static data.

---

## Quick Reference Card

```bash
# Override variables at runtime
ansible-playbook playbook.yml -e "key=value key2=value2"

# Inspect facts before writing conditionals
ansible <host> -m setup -a "filter=ansible_distribution*"

# Run only against hosts matching a condition
ansible-playbook playbook.yml   # when: handles this internally per task

# Throttle loop-heavy playbooks on constrained hosts
ansible-playbook playbook.yml --forks 1
```

| Concept | Key idea |
|---|---|
| Variable precedence | `group_vars/all` < `group_vars/<group>` < `host_vars/<host>` < playbook `vars` < `-e` |
| Facts | Auto-gathered per host; use them instead of hardcoding OS/IP/RAM values |
| `when` | Skips tasks cleanly (`skipping`) on hosts that don't match the condition |
| `loop` | Modern replacement for `with_items`; needs `\| flatten` for nested lists |
| `register` + `debug` | Capture a task's result, then act on or print it in later tasks |
