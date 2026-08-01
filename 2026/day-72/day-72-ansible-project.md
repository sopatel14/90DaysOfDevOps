# Day 72 — Ansible Project: Automate Docker & Nginx Deployment

One command, from a fresh server to a fully running, production-style setup: Docker installed, an app container running, and Nginx serving as a reverse proxy in front of it — all through Ansible roles.

## Architecture

```
Ansible Control Node
        │
        ▼
   Managed Server
   ┌───────────────────────────────┐
   │  Nginx  :80  ──proxy_pass──▶  │
   │                Docker :8080   │
   │                (app container)│
   └───────────────────────────────┘
```

Client hits the server on port **80** → Nginx receives the request → proxies it internally to the container on port **8080** → response flows back through Nginx to the client.

## Project Structure

```
ansible-docker-project/
  ansible.cfg
  inventory.ini
  site.yml                          # Master playbook
  .vault_pass                       # (gitignored)
  group_vars/
    all.yml                         # Common variables
    web/
      vars.yml                      # Nginx variables
      vault.yml                     # Encrypted Docker Hub credentials
  roles/
    common/
      tasks/main.yml
    docker/
      tasks/main.yml
      templates/
        docker-compose.yml.j2
      handlers/main.yml
      defaults/main.yml
    nginx/
      tasks/main.yml
      templates/
        nginx.conf.j2
        app-proxy.conf.j2
      handlers/main.yml
      defaults/main.yml
```

## Key Files

### `site.yml` — Master Playbook

```yaml
---
- name: Apply common configuration
  hosts: all
  become: true
  roles:
    - common
  tags: common

- name: Install Docker and run containers
  hosts: web
  become: true
  roles:
    - docker
  tags: docker

- name: Configure Nginx reverse proxy
  hosts: web
  become: true
  roles:
    - nginx
  tags: nginx
```

### `roles/common/tasks/main.yml`

```yaml
---
- name: Update package cache
  yum:
    update_cache: true
  tags: common

- name: Install common packages
  yum:
    name: "{{ common_packages }}"
    state: present
  tags: common

- name: Set hostname
  hostname:
    name: "{{ inventory_hostname }}"
  tags: common

- name: Set timezone
  timezone:
    name: "{{ timezone }}"
  tags: common

- name: Create deploy user
  user:
    name: deploy
    groups: wheel
    shell: /bin/bash
    state: present
  tags: common
```

### `roles/docker/tasks/main.yml` (core tasks)

```yaml
---
- name: Log in to Docker Hub
  community.docker.docker_login:
    username: "{{ vault_docker_username }}"
    password: "{{ vault_docker_password }}"
  become_user: deploy
  when: vault_docker_username is defined
  tags: docker

- name: Pull application image
  community.docker.docker_image:
    name: "{{ docker_app_image }}"
    tag: "{{ docker_app_tag }}"
    source: pull
  tags: docker

- name: Run application container
  community.docker.docker_container:
    name: "{{ docker_app_name }}"
    image: "{{ docker_app_image }}:{{ docker_app_tag }}"
    state: started
    restart_policy: always
    ports:
      - "{{ docker_app_port }}:{{ docker_container_port }}"
  tags: docker

- name: Wait for container to be healthy
  uri:
    url: "http://localhost:{{ docker_app_port }}"
    status_code: 200
  retries: 5
  delay: 3
  register: health_check
  until: health_check.status == 200
  tags: docker
```

### `roles/nginx/templates/app-proxy.conf.j2` — Reverse Proxy Config

```nginx
# Reverse Proxy to Docker Container -- Managed by Ansible
upstream docker_app {
    server 127.0.0.1:{{ nginx_upstream_port }};
}

server {
    listen {{ nginx_http_port }};
    server_name {{ nginx_server_name }};

    location / {
        proxy_pass http://docker_app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /health {
        access_log off;
        return 200 'OK';
        add_header Content-Type text/plain;
    }
}
```

## Theory: Mapping Concepts to the Week

| Day | Concept | Where it shows up here |
|-----|---------|------------------------|
| 68 | Inventory, ad-hoc commands, SSH setup | `inventory.ini`, `ansible.cfg` connection settings |
| 69 | Playbooks, modules, handlers | `site.yml`, `docker_container` / `docker_image` modules, `Restart Docker` / `Reload Nginx` handlers |
| 70 | Variables, facts, conditionals, loops | `group_vars/all.yml`, `when: vault_docker_username is defined`, `common_packages` list |
| 71 | Roles, templates, Galaxy, Vault | `roles/common`, `roles/docker`, `roles/nginx`; `.j2` templates; `ansible-galaxy init`; `group_vars/web/vault.yml` |
| 72 | Everything combined | `site.yml` orchestrating all three roles in one idempotent run |

**Why each piece matters:**
- **Roles** separate concerns — `common` (baseline OS setup), `docker` (runtime + container), `nginx` (reverse proxy) — so each can be developed, tested, and re-run independently.
- **Handlers** only fire on change (e.g., only reload Nginx if the config template actually changed), which is what keeps repeated runs fast and idempotent.
- **Vault** keeps Docker Hub credentials encrypted at rest in `group_vars/web/vault.yml`, so the repo can be pushed to GitHub without leaking secrets. The `.vault_pass` file decrypts it locally/in CI and is gitignored.
- **Tags** let you touch one layer of the stack without re-running the whole playbook — critical once you're iterating on just the app container or just the Nginx config in a live environment.
- **Idempotency** is the real test of a good playbook: running `site.yml` a second time should report mostly `ok` and little-to-no `changed`, proving the system converges to the same state regardless of how many times you apply it.

## Deployment

```bash
# Dry run first — always
ansible-playbook site.yml --check --diff

# Full deploy
ansible-playbook site.yml
```

### Selective runs with tags

```bash
# Only Docker + containers
ansible-playbook site.yml --tags docker

# Only Nginx config
ansible-playbook site.yml --tags nginx

# Skip common baseline setup
ansible-playbook site.yml --skip-tags common
```

## Verification & Screenshots


|---|---|
| Full end-to-end playbook run | `ansible yml -1` |
| `docker ps` on the managed node showing the running container with correct port mapping | `docker ps – app server` |
| Container responding directly on 8080 | `curl on app server 8080` |
| **Nginx reverse-proxying on port 80** 
| Tag-based selective run — Docker only | `tags - docker` |
| Tag-based selective run — Nginx only | `tags - nginx` |
| Tag-based selective run — common only | `tags - common` |

> Insert each image below its row once exported, e.g.:
> `![docker ps](./screenshots/docker-ps-app-server.png)`

<img width="2212" height="2030" alt="image" src="https://github.com/user-attachments/assets/2724bd76-c3dd-456d-88bd-5d8201001578" />
<img width="2112" height="1028" alt="image" src="https://github.com/user-attachments/assets/e201ecb4-98bf-437c-88ec-a889fc8b378b" />
<img width="1454" height="214" alt="image" src="https://github.com/user-attachments/assets/6fc99c5e-edad-4180-a1c5-bde86ff4f90c" />
<img width="842" height="430" alt="image" src="https://github.com/user-attachments/assets/0e75d1b8-78de-4628-a79b-454bb3f68bee" />
<img width="1026" height="336" alt="image" src="https://github.com/user-attachments/assets/60531b44-15a8-4efd-80d0-8003ab350796" />
<img width="1298" height="1764" alt="image" src="https://github.com/user-attachments/assets/a4a4dcf1-ae24-416a-800d-0d2bd0a7f33b" />
<img width="1760" height="998" alt="image" src="https://github.com/user-attachments/assets/691aade6-eba6-4a88-9eee-ef5240224399" />
<img width="1306" height="2060" alt="image" src="https://github.com/user-attachments/assets/a0a46b09-88fd-47dc-8411-c971b675daa9" />
<img width="1258" height="644" alt="image" src="https://github.com/user-attachments/assets/f7a1a46d-fb7f-4d97-a75e-366bd4aea222" />


## How Tags Were Used

Each role's tasks are tagged (`common`, `docker`, `nginx`), which let the full stack be deployed once with `site.yml`, and then iterated on selectively — e.g., pushing an Nginx config change with `--tags nginx` without touching Docker or re-running baseline package installs.

## How Vault Protected Docker Hub Credentials

Docker Hub username and token are stored in `group_vars/web/vault.yml`, encrypted with `ansible-vault create`. The plaintext values never touch the repo — `ansible.cfg` points to a local `vault_password_file` (`.vault_pass`, gitignored) so Ansible can decrypt at run time, while anyone browsing the GitHub repo only sees ciphertext.
