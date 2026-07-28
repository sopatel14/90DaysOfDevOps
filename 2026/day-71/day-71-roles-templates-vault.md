# Day 71 - Roles, Galaxy, Templates and Vault

## Overview

As playbooks grow, keeping everything in a single YAML file becomes unmanageable. Today's focus was on organizing Ansible automation the way it's done in real infrastructure: **Roles** for reusable, structured automation; **Jinja2 Templates** for dynamic configuration files; **Ansible Galaxy** for sharing and reusing community-built roles; and **Ansible Vault** for keeping secrets like passwords and API keys out of plain text.

By the end of this task, all four pieces were combined into a single `site.yml` that configures web, app, and database servers in one run.

---

## Task 1: Jinja2 Templates

### What are Jinja2 Templates?

Jinja2 templates let Ansible generate configuration files dynamically instead of copying static files. A template can reference:

- **Variables** — `{{ variable }}`
- **Facts** — data Ansible automatically gathers about a host, e.g. `{{ ansible_hostname }}`
- **Filters** — transform or provide fallback values, e.g. `{{ variable | default('default') }}`
- **Conditionals** — `{% if %} ... {% endif %}`
- **Loops** — `{% for %} ... {% endfor %}`

Templates use the `.j2` extension by convention and are deployed with the `template` module (as opposed to the `copy` module, which is for static files).

### Template Created
**File:** `templates/nginx-vhost.conf.j2`

```jinja2
# Managed by Ansible -- do not edit manually
server {
    listen {{ http_port | default(80) }};
    server_name {{ ansible_hostname }};

    root /var/www/{{ app_name }};
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    access_log /var/log/nginx/{{ app_name }}_access.log;
    error_log /var/log/nginx/{{ app_name }}_error.log;
}
```

Key elements used:
- `{{ http_port }}` — configurable port, defaults to 80 if undefined
- `{{ ansible_hostname }}` — server's hostname pulled from gathered facts
- `{{ app_name }}` — application name, passed in as a variable

### Playbook: `template-demo.yml`
- Installs Nginx using the `yum` module
- Creates the web root directory
- Deploys the vhost config from the template
- Deploys a dynamic `index.html` page
- Notifies a handler to restart Nginx whenever the config changes

Run with `--diff` so the rendered output is visible in the console:

```bash
ansible-playbook template-demo.yml --diff
```

### Verification
The generated config at `/etc/nginx/conf.d/terraweek-app.conf` confirmed correct rendering:
- Port: `80`
- Server name: `[actual hostname]`
- Root: `/var/www/terraweek-app`

### Screenshots

<img width="1688" height="2100" alt="image" src="https://github.com/user-attachments/assets/8370b87c-6814-4f09-8298-77f1e103acaa" />


<img width="1112" height="660" alt="image" src="https://github.com/user-attachments/assets/19b4035b-5988-41f6-8146-7107d12e1bcc" />


---

## Task 2: Understanding the Role Structure

### What is a Role?
A role is a standardized directory structure that packages tasks, handlers, templates, files, and variables into one reusable, shareable unit. Instead of one long playbook, a role can simply be called by name from any playbook.

```
roles/
  webserver/
    tasks/
      main.yml         # The main task list
    handlers/
      main.yml         # Handlers (restart services, etc.)
    templates/
      nginx.conf.j2    # Jinja2 templates
    files/
      index.html       # Static files to copy
    vars/
      main.yml         # Role variables (high priority)
    defaults/
      main.yml         # Default variables (low priority, easily overridden)
    meta/
      main.yml         # Role metadata and dependencies
```

Every directory's `main.yml` is loaded automatically by Ansible — there's no need to explicitly `include` it. Only the directories actually needed have to be created; unused ones can be deleted.

### vars vs defaults

| Feature | vars/main.yml | defaults/main.yml |
|---------|--------------|-------------------|
| Priority | High | Low |
| Override | Difficult (needs `-e` or higher-precedence source) | Easy (overridden by inventory, play vars, or `-e`) |
| Use Case | Internal, fixed configuration | User-facing, tunable settings |
| Example | Hardcoded internal paths | Configurable ports, feature flags |

In short: `defaults/` is meant to be overridden by whoever calls the role; `vars/` is not.

### Generated Role Skeleton
Created with:
```bash
ansible-galaxy init roles/webserver
```

> `![tree output](screenshots/task2-tree-output.png)`
> *tree roles/webserver showing the full generated skeleton*

### Key Directories
- **tasks/** — what the role actually does (the main task list)
- **handlers/** — actions triggered by `notify` (e.g. restarting a service)
- **templates/** — dynamic `.j2` configuration files
- **files/** — static files copied as-is
- **vars/** — high-priority, role-internal variables
- **defaults/** — low-priority variables meant to be overridden by callers
- **meta/** — role metadata, author info, and dependencies on other roles

---

## Task 3: Build a Custom Webserver Role

### Role Structure Created
```
roles/webserver/
├── defaults/main.yml   # Default variables
├── handlers/main.yml   # Service handlers
├── tasks/main.yml      # Main tasks
└── templates/
    ├── nginx.conf.j2   # Main Nginx config
    ├── vhost.conf.j2   # Virtual host config
    └── index.html.j2   # Web page template
```

### Variables (`defaults/main.yml`)
```yaml
---
http_port: 80
app_name: myapp
max_connections: 512
```
- `http_port`: 80 (default, unused override in this run)
- `app_name`: overridden to `terraweek` when the role is called
- `max_connections`: 512 (default)

### Tasks (`tasks/main.yml`)
Installs Nginx, deploys the main config and vhost config from templates, creates the web root, deploys the index page, and starts/enables the service — with handlers notified on any config change.

### Handlers (`handlers/main.yml`)
```yaml
---
- name: Restart Nginx
  service:
    name: nginx
    state: restarted
```

### Templates Created
1. **nginx.conf.j2** — main Nginx configuration; uses `max_connections` to set worker connections
2. **vhost.conf.j2** — virtual host config; uses `http_port`, `app_name`, and the `ansible_hostname` fact
3. **index.html.j2** — web page template; shows app name, hostname, IP, and `app_env` (with a `default('development')` fallback)

### Playbook: `site.yml`
Calls the webserver role with:
```yaml
roles:
  - role: webserver
    vars:
      app_name: terraweek
      http_port: 80
```

Run with:
```bash
ansible-playbook site.yml
```

### Verification
Curling the web server confirmed the custom `terraweek` page loaded correctly, rendered with the live hostname and IP.

### Screenshots

<img width="1710" height="1124" alt="image" src="https://github.com/user-attachments/assets/8199c619-6a4f-499b-8004-1186e0d772bb" />


<img width="1146" height="116" alt="image" src="https://github.com/user-attachments/assets/0a7a3038-06ec-427e-9388-f5bae9edf9bb" />


<img width="864" height="302" alt="image" src="https://github.com/user-attachments/assets/0210e70f-781c-4118-99f0-74f23110cdeb" />


---

## Task 4: Ansible Galaxy — Community Roles

### What is Ansible Galaxy?
Ansible Galaxy is a public repository of community-contributed roles — conceptually similar to Docker Hub for containers, but for automation roles. Instead of writing a role from scratch for common tasks (installing Docker, setting up NTP, configuring MySQL), an existing, tested role can be pulled in directly.

### Searched Roles
```bash
ansible-galaxy search nginx --platforms EL
ansible-galaxy search mysql
```
- `nginx` — various community web server roles
- `mysql` — various community database roles

### Installed Role
**Role:** `geerlingguy.docker`
**Purpose:** Installs and configures Docker

```bash
ansible-galaxy install geerlingguy.docker
ansible-galaxy list
```

### Playbook: `docker-setup.yml`
Uses the `geerlingguy.docker` role to install Docker, configure the Docker service, and enable it on boot — with a single `roles:` entry instead of manually writing installation tasks.

### requirements.yml — Theory
`requirements.yml` lists external role dependencies in one file instead of installing each role manually via separate commands.

```yaml
---
roles:
  - name: geerlingguy.docker
    version: "7.4.1"
  - name: geerlingguy.ntp
```

```bash
ansible-galaxy install -r requirements.yml
```

**Why use `requirements.yml` instead of installing roles manually?**
- **Version Control / Pinning** — pins exact role versions, so an upstream update can't silently break a playbook
- **Reproducibility** — every teammate and every environment installs the identical set of roles
- **Team Collaboration** — the dependency list is committed to Git and shared automatically, instead of relying on everyone remembering which `ansible-galaxy install` commands to run
- **CI/CD Integration** — a pipeline can run one command (`ansible-galaxy install -r requirements.yml`) to provision all dependencies automatically
- **Documentation** — the file itself serves as a manifest of every external role the project depends on

### Screenshots

<img width="1192" height="144" alt="image" src="https://github.com/user-attachments/assets/12d2ccad-6e4e-46a1-9be6-a5bc5a260fb9" />


<img width="1658" height="2050" alt="image" src="https://github.com/user-attachments/assets/d54111d4-d89a-4298-8b48-f901d23b1fac" />


---

## Task 5: Ansible Vault — Encrypting Secrets

### What is Ansible Vault?
Ansible Vault encrypts sensitive data — passwords, API keys, tokens — so secrets can be safely committed to version control alongside the rest of the automation code, instead of living in plain text.

### Commands Learned

| Command | Purpose |
|---------|---------|
| `ansible-vault create` | Create a new encrypted file |
| `ansible-vault edit` | Edit an existing encrypted file |
| `ansible-vault view` | View decrypted contents without saving them to disk |
| `ansible-vault encrypt` | Encrypt an existing plain-text file |
| `ansible-vault decrypt` | Permanently decrypt a file back to plain text |

### Vault File Created
**File:** `group_vars/db/vault.yml`
**Variables:**
```yaml
vault_db_password: SuperSecretP@ssw0rd
vault_db_root_password: R00tP@ssw0rd123
vault_api_key: sk-abc123xyz789
```

Created with:
```bash
ansible-vault create group_vars/db/vault.yml
```
Viewing the raw file with `cat` shows only encrypted, unreadable ciphertext — confirming the secrets are protected at rest.

### Working with Vault
1. **Create** — `ansible-vault create file.yml`
2. **View** — `ansible-vault view file.yml`
3. **Edit** — `ansible-vault edit file.yml`
4. **Encrypt** (existing plain-text file) — `ansible-vault encrypt file.yml`

Vault-encrypted files behave like normal YAML once decrypted at runtime — Ansible handles the decryption transparently during playbook execution, so tasks reference `vault_db_password` etc. exactly like any other variable.

### Using Vault Variables in a Playbook
`db-setup.yml` references `vault_db_password` in a `debug` task (only checking its length, never printing the actual secret — printing secrets in a real environment should always be avoided).

```bash
ansible-playbook db-setup.yml --ask-vault-pass
```

### Authentication Methods

| Method | Use Case |
|--------|----------|
| `--ask-vault-pass` | Manual, interactive runs |
| `--vault-password-file` | CI/CD pipelines and automation |
| `ansible.cfg` (`vault_password_file`) | Permanent, project-wide configuration |

**Why is `--vault-password-file` better than `--ask-vault-pass` for automated pipelines?**
`--ask-vault-pass` requires a human to type the password interactively, which is impossible in an unattended CI/CD job. `--vault-password-file` points to a file (or an executable script that returns the password) so the pipeline can decrypt secrets non-interactively. The password file itself is kept out of version control (via `.gitignore`) and restricted with file permissions, so automation gets non-interactive access without hardcoding the password inside a script or committing it anywhere.

### Security Best Practices
- Never commit `.vault_pass` to Git
- Add password files to `.gitignore`
- Restrict file permissions: `chmod 600 .vault_pass`
- Use strong, unique vault passwords
- Prefer `ansible-vault encrypt_string` when only a single value needs encrypting inline, rather than encrypting an entire file

### Screenshots

<img width="1236" height="416" alt="image" src="https://github.com/user-attachments/assets/b892c641-db01-4ba3-896f-0a0d4ca965fd" />

<img width="1428" height="584" alt="image" src="https://github.com/user-attachments/assets/4558c6ce-8de7-4c46-ad04-c3eae66729b0" />


---

## Task 6: Combined Implementation

### Complete Infrastructure as Code
This task ties together everything learned today into a single run:

1. **Roles** — the `webserver` role configures web servers
2. **Galaxy** — `geerlingguy.docker` configures app servers
3. **Templates** — `db-config.j2` renders database configuration
4. **Vault** — encrypted database credentials are injected into that config

### `site.yml` Overview
```yaml
---
- name: Configure web servers
  hosts: web
  become: true
  roles:
    - role: webserver
      vars:
        app_name: terraweek
        http_port: 80

- name: Configure app servers with Docker
  hosts: app
  become: true
  roles:
    - geerlingguy.docker

- name: Configure database servers
  hosts: db
  become: true
  tasks:
    - name: Create DB config with secrets
      template:
        src: templates/db-config.j2
        dest: /etc/db-config.env
        owner: root
        mode: '0600'
```

### Database Template: `db-config.j2`
```jinja2
# Database Configuration -- Managed by Ansible
DB_HOST={{ ansible_default_ipv4.address }}
DB_PORT={{ db_port | default(3306) }}
DB_PASSWORD={{ vault_db_password }}
DB_ROOT_PASSWORD={{ vault_db_root_password }}
```
Uses:
- `ansible_default_ipv4.address` — the host's IP, from gathered facts
- `db_port` — customizable, defaults to 3306
- `vault_db_password` / `vault_db_root_password` — pulled from the encrypted vault file

### Security Implementation
- Template rendered with `mode: '0600'` — only root can read or write the file
- Secrets are injected directly from Vault at runtime, never hardcoded in the playbook or template

### Verification
- **Web server** — the custom `terraweek` page loads via curl
- **App server** — Docker is installed and running (`docker --version`)
- **Database server** — `/etc/db-config.env` exists with the correct values and `600` permissions

### Screenshots

<img width="1488" height="2032" alt="image" src="https://github.com/user-attachments/assets/1cd6d9ac-11c6-4c9b-b8c7-4d4fdb5c4775" />

<img width="828" height="238" alt="image" src="https://github.com/user-attachments/assets/bd8dcb71-2cbf-49bc-b69f-3e022d977ea3" />

---

## Roles vs Playbooks vs Ad-hoc Commands

| Approach | When to Use |
|----------|-------------|
| **Ad-hoc command** (`ansible web -m yum -a "name=nginx state=present"`) | One-off, throwaway actions — a quick check or fix on a handful of hosts, not meant to be repeated or version-controlled |
| **Playbook** | A specific, self-contained set of tasks for one project — e.g. "deploy this one app," when the logic isn't reused elsewhere |
| **Role** | Reusable automation that will be called across multiple playbooks or projects — e.g. "configure any web server," "install Docker on any host" — anything that benefits from being structured, shared, and version-pinned via Galaxy |

In practice: ad-hoc commands are for quick exploration, playbooks are for orchestrating a specific deployment, and roles are for packaging the reusable building blocks that playbooks call.

---

## Summary

- **Jinja2 templates** turn static config files into dynamic ones driven by variables and facts, with `.j2` as the naming convention and `default()` as a safety net for undefined variables.
- **Roles** impose a standard directory structure (`tasks/`, `handlers/`, `templates/`, `files/`, `vars/`, `defaults/`, `meta/`) so automation is reusable instead of duplicated across playbooks.
- **`defaults/` vs `vars/`** boils down to override intent: defaults are meant to be overridden, vars are not.
- **Ansible Galaxy** turns hours of writing common roles from scratch into a single `ansible-galaxy install`, with `requirements.yml` making that reproducible across a whole team or CI pipeline.
- **Ansible Vault** keeps secrets encrypted at rest while remaining fully usable at runtime, with `--vault-password-file` being the practical choice for automation over the interactive `--ask-vault-pass`.
- Combining all four in one `site.yml` mirrors how real infrastructure is actually managed: reusable roles, dynamic templates, external community roles, and encrypted secrets working together.

---

## Resources
- [Ansible Roles Documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html)
- [Jinja2 Templating in Ansible](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_templating.html)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [Ansible Vault Documentation](https://docs.ansible.com/ansible/latest/vault_guide/index.html)
- [geerlingguy.docker Role](https://galaxy.ansible.com/geerlingguy/docker)
