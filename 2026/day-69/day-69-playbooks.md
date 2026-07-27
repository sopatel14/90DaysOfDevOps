# Day 69 — Ansible Playbooks and Modules

## 1. First Playbook — `install-nginx.yml`

```yaml
---
- name: Install and start Nginx on web servers   # PLAY — targets the "web" group
  hosts: web                                     # Inventory group to run against
  become: true                                   # Run tasks as root (sudo)

  tasks:                                         # List of TASKS in this play
    - name: Install Nginx                        # TASK — one unit of work
      yum:                                        # MODULE — what Ansible actually does
        name: nginx
        state: present                           # install if not already installed

    - name: Start and enable Nginx
      service:
        name: nginx
        state: started                           # start if not running
        enabled: true                            # start on boot

    - name: Create a custom index page
      copy:
        content: "<h1>Deployed by Ansible - TerraWeek Server</h1>"
        dest: /usr/share/nginx/html/index.html
```

> Use `apt` instead of `yum` if the target instances run Ubuntu/Debian.

Run it:

```bash
ansible-playbook install-nginx.yml
```

<img width="2052" height="752" alt="image" src="https://github.com/user-attachments/assets/bae3326a-288b-49f6-a010-fc1115a8236d" />

Run it again — the second run should show `ok` instead of `changed` for every task. That's **idempotency**: Ansible only makes a change when the actual state differs from the desired state.

<img width="2040" height="676" alt="image" src="https://github.com/user-attachments/assets/af40c08f-4e54-4257-af63-b40390c321b1" />

**Verification:** `curl` the web server's public IP and confirm the custom index page loads.

---

## 2. Playbook Structure — Answers

| Question | Answer |
|---|---|
| Difference between a **play** and a **task**? | A play targets a group of hosts and defines *how* to configure them (become, vars, handlers, etc.). A task is a single action inside a play — one module call with its arguments. |
| Can a playbook have multiple plays? | Yes. A single `.yml` file can contain any number of plays, each starting with its own `- name:` / `hosts:` block, each targeting a different group. |
| `become: true` at the **play** level vs the **task** level? | At the play level it applies to *every* task in that play by default. At the task level it overrides the play-level setting for just that one task (e.g. one task needs sudo, the rest don't). |
| What happens if a task fails? | By default Ansible stops executing further tasks on that host and marks it failed (other unaffected hosts continue). Setting `ignore_errors: yes` on a task lets the play continue past that failure. |

---

## 3. Essential Modules — `essential-modules.yml`

| Module | Purpose |
|---|---|
| `yum` / `apt` | Install, remove, or update packages |
| `service` | Start, stop, restart, or enable/disable a service |
| `copy` | Transfer a file (or inline content) from the control node to managed nodes |
| `file` | Create directories/symlinks, remove paths, set ownership & permissions |
| `command` | Run a command with **no** shell features (no pipes, redirects, env expansion) |
| `shell` | Run a command **with** full shell features (pipes, redirects, `$VAR`) |
| `lineinfile` | Add, modify, or remove a single line in an existing file |
| `debug` / `register` | Capture a task's output into a variable, then print it |

```yaml
---
- name: Practice essential modules
  hosts: all
  become: true

  tasks:
    - name: Install multiple packages
      yum:
        name: [git, curl, wget, tree]
        state: present

    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: true

    - name: Copy config file
      copy:
        src: files/app.conf
        dest: /etc/app.conf
        owner: root
        group: root
        mode: '0644'

    - name: Create application directory
      file:
        path: /opt/myapp
        state: directory
        owner: ec2-user
        mode: '0755'

    - name: Check disk space
      command: df -h
      register: disk_output

    - name: Print disk space
      debug:
        var: disk_output.stdout_lines

    - name: Count running processes
      shell: ps aux | wc -l
      register: process_count

    - name: Show process count
      debug:
        msg: "Total processes: {{ process_count.stdout }}"

    - name: Set timezone in environment
      lineinfile:
        path: /etc/environment
        line: 'TZ=Asia/Kolkata'
        create: true
```

A `files/app.conf` sample file sits alongside the playbook so the `copy` task has something to transfer.

### `command` vs `shell`

| Feature | `command` | `shell` |
|---|---|---|
| Pipes / redirects / env vars | ❌ No | ✅ Yes |
| Security | ✅ Safer (no injection risk) | ⚠️ Riskier — avoid interpolating untrusted input |
| When to use | Simple, single commands (`df -h`, `uname -a`) | Anything needing pipes, redirects, or shell expansion (`ps aux \| wc -l`) |

**Rule of thumb:** default to `command`; reach for `shell` only when the task genuinely needs shell syntax.

---

## 4. Handlers — `nginx-config.yml`

Handlers are tasks that only run when a task **notifies** them, and they run **once**, at the end of the play, even if notified by several tasks.

```yaml
---
- name: Configure Nginx with a custom config
  hosts: web
  become: true

  tasks:
    - name: Install Nginx
      yum:
        name: nginx
        state: present

    - name: Deploy Nginx config
      copy:
        src: files/nginx.conf
        dest: /etc/nginx/nginx.conf
        owner: root
        mode: '0644'
      notify: Restart Nginx

    - name: Deploy custom index page
      copy:
        content: "<h1>Managed by Ansible</h1><p>Server: {{ inventory_hostname }}</p>"
        dest: /usr/share/nginx/html/index.html

    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: true

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

### Before / after comparison

**First run** — the config file is new, so the `copy` task reports `changed`, which triggers the handler:

<img width="2064" height="1624" alt="image" src="https://github.com/user-attachments/assets/a9edf67a-f003-4675-888a-e011f83f449b" />


```text
TASK [Deploy Nginx config] ******************************
changed: [web-server]          ← config changed, handler notified

RUNNING HANDLER [Restart Nginx] *************************
changed: [web-server]          ← handler fires

PLAY RECAP ************************************************
web-server : ok=6  changed=2  unreachable=0  failed=0
```

**Second run** — nothing changed, so the handler is skipped entirely:

```text
TASK [Deploy Nginx config] ******************************
ok: [web-server]               ← no change, no notification

PLAY RECAP ************************************************
web-server : ok=6  changed=0  unreachable=0  failed=0
```

**Answer:** No — the handler runs only on the first run. On the second run nothing changed, so `notify` never fires and the handler is silently skipped, avoiding an unnecessary restart.

---

## 5. Dry Run, Diff, and Verbosity

```bash
# Dry run — shows what WOULD change, changes nothing
ansible-playbook install-nginx.yml --check

# Diff mode — shows actual file content differences
ansible-playbook nginx-config.yml --check --diff

# Verbosity levels
ansible-playbook install-nginx.yml -v      # task results
ansible-playbook install-nginx.yml -vv     # + module args/output
ansible-playbook install-nginx.yml -vvv    # + SSH/connection debugging

# Limit to specific hosts
ansible-playbook install-nginx.yml --limit web-server

# Preview scope without running
ansible-playbook install-nginx.yml --list-hosts
ansible-playbook install-nginx.yml --list-tasks
```

| Flag | Shows |
|---|---|
| `--check` | Dry run — predicts `changed`/`ok` per task, makes zero real changes |
| `--diff` | The literal before/after content of files that would be modified |
| `-v` / `-vv` / `-vvv` | Increasing levels of debug detail (task result → module args → SSH handshake) |

**Why `--check --diff` matters most for production:** together they let you see *exactly* which tasks would change something and *exactly* what the file content would become — before touching a live system. It's the closest thing Ansible has to a safe preview/rollback check, catching unintended changes (wrong file, bad template, wrong host group) before they happen.

---

## 6. Multiple Plays — `multi-play.yml`

```yaml
---
- name: Configure web servers
  hosts: web
  become: true
  tasks:
    - name: Install Nginx
      yum:
        name: nginx
        state: present
    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: true

- name: Configure app servers
  hosts: app
  become: true
  tasks:
    - name: Install Node.js dependencies
      yum:
        name: [gcc, make]
        state: present
    - name: Create app directory
      file:
        path: /opt/app
        state: directory
        mode: '0755'

- name: Configure database servers
  hosts: db
  become: true
  tasks:
    - name: Install MySQL client
      yum:
        name: mysql
        state: present
    - name: Create data directory
      file:
        path: /var/lib/appdata
        state: directory
        mode: '0700'
```

```bash
ansible-playbook multi-play.yml
```

Each play runs sequentially and only against its own `hosts:` group:

```text
PLAY [Configure web servers] ***************************
TASK [Install Nginx] ************************** ok: [web-server]
TASK [Start Nginx] ****************************** ok: [web-server]

PLAY [Configure app servers] ***************************
TASK [Install Node.js dependencies] ******** changed: [app-server]
TASK [Create app directory] **************** changed: [app-server]

PLAY [Configure database servers] ***********************
TASK [Install MySQL client] **************** changed: [db-server]
TASK [Create data directory] *************** changed: [db-server]

PLAY RECAP ************************************************
app-server : ok=3  changed=2
db-server  : ok=3  changed=2
web-server : ok=3  changed=0
```

**Verification:** Nginx only ends up on `web` hosts, MySQL only on `db` hosts, and Node build tools only on `app` hosts — confirmed by the recap showing each group's task set applied to only its own hosts.

---

## Quick Reference Card

```bash
# Validate before running
ansible-playbook playbook.yml --syntax-check

# Preview
ansible-playbook playbook.yml --check --diff
ansible-playbook playbook.yml --list-hosts
ansible-playbook playbook.yml --list-tasks

# Run
ansible-playbook playbook.yml
ansible-playbook playbook.yml --limit web-server-01

# Debug
ansible-playbook playbook.yml -vvv
```

| State value | Meaning |
|---|---|
| `state: present` | Install if not already installed |
| `state: absent` | Remove |
| `state: started` | Start if not running |
| `state: restarted` | Always restart |

**Key takeaways**
- `notify` + `handlers` avoid unnecessary service restarts — the handler runs once, only when triggered.
- `register` captures a task's output; `debug` prints it.
- `{{ inventory_hostname }}` is a built-in variable holding the current host's name.
- Always run `--syntax-check` → `--check --diff` → `--limit <canary-host>` → full run, in that order.
