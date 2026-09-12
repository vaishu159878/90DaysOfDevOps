# Day 69 — Ansible Playbooks and Modules 🚀

Today I moved from Ansible ad-hoc commands to **Ansible Playbooks**.

Ad-hoc commands are useful for quick tasks, but playbooks make it possible to define the desired state of servers and automate the same configuration repeatedly.

The main thing I focused on today was **idempotency** — running the same playbook again should not make unnecessary changes.

---

## 🛠️ Lab Environment

- AWS EC2
- Ubuntu Linux
- Ansible
- SSH
- Nginx
- YAML

My Ansible control node connects to the managed EC2 servers through SSH.

```text
                  Ansible Control Node
                         |
                         | SSH
          +--------------+--------------+
          |              |              |
          ▼              ▼              ▼
       Web Server     App Server     DB Server
          |              |              |
        Nginx         App setup      MySQL Client
```

---

# 1. Ansible Inventory

I created an inventory file to define the servers that Ansible should manage.

Example:

```ini
[web]
web-server ansible_host=<WEB_PRIVATE_IP>

[app]
app-server ansible_host=<APP_PRIVATE_IP>

[db]
db-server ansible_host=<DB_PRIVATE_IP>
```

I also configured the Ubuntu user and SSH private key.

Then tested connectivity:

```bash
ansible all -i inventory.ini -m ping
```

Expected result:

```text
web-server | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

The `ping` module confirmed that Ansible could successfully connect to my managed servers.

---

# 2. First Playbook — Install Nginx

I created:

```text
install-nginx.yml
```

The playbook installs Nginx, starts the service, enables it at boot, and creates a custom web page.

```yaml
---
- name: Install and configure Nginx on web servers
  hosts: web
  become: true

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: true

    - name: Start and enable Nginx
      service:
        name: nginx
        state: started
        enabled: true

    - name: Create custom index page
      copy:
        content: "<h1>Deployed by Ansible - TerraWeek Server</h1>"
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: '0644'
```

I used `apt` because my managed servers are running Ubuntu.

Before executing the playbook, I checked the syntax:

```bash
ansible-playbook -i inventory.ini install-nginx.yml --syntax-check
```

Then ran:

```bash
ansible-playbook -i inventory.ini install-nginx.yml
```

---

# 3. Understanding Playbooks

A playbook contains one or more **plays**.

A play defines:

- Which hosts should be targeted
- Privilege escalation requirements
- Tasks that should be executed

A task defines one specific unit of work.

Example:

```yaml
- name: Install Nginx
  apt:
    name: nginx
    state: present
```

Here:

- `Install Nginx` → Task name
- `apt` → Ansible module
- `name` and `state` → Module arguments

The basic structure is:

```text
Playbook
   │
   ├── Play
   │    │
   │    ├── Task
   │    │    └── Module
   │    │
   │    └── Task
   │         └── Module
   │
   └── Play
        │
        └── Tasks
```

---

# 4. What does `become: true` do?

I used:

```yaml
become: true
```

to allow Ansible to perform tasks with elevated privileges.

For example, installing packages and modifying files under `/etc` generally requires root privileges.

It can be defined at the play level:

```yaml
become: true
```

which applies to tasks in that play.

It can also be defined for an individual task when privilege escalation is needed only there.

---

# 5. Idempotency

One of the most important concepts I learned today was **idempotency**.

I ran my Nginx playbook once:

```bash
ansible-playbook -i inventory.ini install-nginx.yml
```

The first run showed tasks as:

```text
changed
```

Then I ran the exact same playbook again:

```bash
ansible-playbook -i inventory.ini install-nginx.yml
```

The second run showed:

```text
ok
```

instead of making the same changes again.

Example:

```text
First run:

TASK [Install Nginx]
changed: [web-server]

TASK [Start and enable Nginx]
changed: [web-server]


Second run:

TASK [Install Nginx]
ok: [web-server]

TASK [Start and enable Nginx]
ok: [web-server]
```

This demonstrated Ansible's idempotent approach.

Instead of blindly executing commands again, Ansible checks the current state and changes the server only when necessary.

---

# 6. Verify Nginx

After running the playbook, I verified the service:

```bash
ansible web -i inventory.ini -b -m command -a "systemctl is-active nginx"
```

Expected:

```text
active
```

I also verified the web page using:

```bash
curl http://<WEB_SERVER_PUBLIC_IP>
```

Expected:

```html
<h1>Deployed by Ansible - TerraWeek Server</h1>
```

---

# 7. Essential Ansible Modules

I practiced seven commonly used modules.

## `apt`

Used to install and remove packages on Ubuntu/Debian systems.

```yaml
- name: Install required packages
  apt:
    name:
      - git
      - curl
      - wget
      - tree
    state: present
    update_cache: true
```

---

## `service`

Used to manage services.

```yaml
- name: Ensure Nginx is running
  service:
    name: nginx
    state: started
    enabled: true
```

---

## `copy`

Used to copy files from the Ansible control node to managed servers.

```yaml
- name: Copy application configuration
  copy:
    src: files/app.conf
    dest: /etc/app.conf
    owner: root
    group: root
    mode: '0644'
```

---

## `file`

Used to create directories and manage file permissions.

```yaml
- name: Create application directory
  file:
    path: /opt/myapp
    state: directory
    owner: ubuntu
    group: ubuntu
    mode: '0755'
```

---

## `command`

Used to execute commands without going through a shell.

```yaml
- name: Check disk space
  command: df -h
  register: disk_output
```

I used `register` to save the output.

Then used `debug` to display it:

```yaml
- name: Print disk space
  debug:
    var: disk_output.stdout_lines
```

---

## `shell`

Used when shell features are required.

For example, I used a pipe:

```yaml
- name: Count running processes
  shell: ps aux | wc -l
  register: process_count
```

Then:

```yaml
- name: Show process count
  debug:
    msg: "Total processes: {{ process_count.stdout }}"
```

---

## `lineinfile`

Used to ensure a particular line exists in a file.

```yaml
- name: Set timezone environment variable
  lineinfile:
    path: /etc/environment
    line: 'TZ=Asia/Kolkata'
    create: true
```

---

# 8. `command` vs `shell`

This was another important difference I practiced.

### `command`

```yaml
command: df -h
```

Runs the command directly and does not provide normal shell features.

### `shell`

```yaml
shell: ps aux | wc -l
```

Runs through a shell and supports features such as:

- Pipes
- Redirects
- Shell operators
- Shell expansion

### My understanding

I would prefer `command` when I don't need shell functionality.

I would use `shell` only when shell features are actually required.

---

# 9. Handlers

Next, I learned how Ansible handlers can prevent unnecessary service restarts.

I created:

```text
nginx-config.yml
```

The configuration task contains:

```yaml
notify: Restart Nginx
```

The handler is:

```yaml
handlers:

  - name: Restart Nginx
    service:
      name: nginx
      state: restarted
```

The flow is:

```text
Nginx configuration changes
          ↓
        notify
          ↓
   Restart Nginx handler
          ↓
      Nginx restarts
```

If the configuration doesn't change:

```text
Nginx configuration unchanged
          ↓
       No notify
          ↓
    No service restart
```

---

# 10. Handler Test

On the first run, the configuration file was deployed:

```text
TASK [Deploy Nginx configuration]
changed: [web-server]

RUNNING HANDLER [Restart Nginx]
changed: [web-server]
```

On the second run:

```text
TASK [Deploy Nginx configuration]
ok: [web-server]
```

The handler did not run because there was no change to the configuration.

This helped me understand why handlers are useful in configuration management.

---

# 11. Check Mode

Before applying changes, I practiced Ansible's check mode:

```bash
ansible-playbook -i inventory.ini install-nginx.yml --check
```

Check mode allows me to preview what Ansible expects to change without intentionally applying those changes.

---

# 12. Diff Mode

I also used:

```bash
ansible-playbook -i inventory.ini nginx-config.yml --check --diff
```

`--diff` is particularly useful when managing configuration files because it can show the difference between the existing file and the desired content where supported.

Using:

```bash
--check --diff
```

gives me a safer way to review configuration changes before applying them.

---

# 13. Verbosity

For troubleshooting, I practiced different verbosity levels.

```bash
ansible-playbook -i inventory.ini install-nginx.yml -v
```

```bash
ansible-playbook -i inventory.ini install-nginx.yml -vv
```

```bash
ansible-playbook -i inventory.ini install-nginx.yml -vvv
```

Higher verbosity provides more information about what Ansible is doing and is useful when debugging connection or execution problems.

---

# 14. Limiting Execution

I also learned how to run a playbook against a specific host.

```bash
ansible-playbook -i inventory.ini install-nginx.yml --limit web-server
```

This is useful when I don't want to execute a playbook against every matching server.

---

# 15. Multiple Plays

Finally, I created:

```text
multi-play.yml
```

The playbook contains separate plays for:

```text
Web Servers
    ↓
Nginx

App Servers
    ↓
Application dependencies + app directory

DB Servers
    ↓
MySQL client + data directory
```

Example structure:

```yaml
- name: Configure web servers
  hosts: web
  become: true
  tasks:
    ...

- name: Configure app servers
  hosts: app
  become: true
  tasks:
    ...

- name: Configure database servers
  hosts: db
  become: true
  tasks:
    ...
```

This demonstrated that a single Ansible playbook can contain multiple plays targeting different inventory groups.

---
