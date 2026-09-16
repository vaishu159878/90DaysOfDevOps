# Day 71 — Ansible Roles, Jinja2 Templates, Galaxy and Vault


## 1. Ansible Controller Setup

I used an Ubuntu EC2 instance as my Ansible controller and configured three target servers:

```text
web-server
app-server
db-server
```

I first tested Ansible connectivity:

```bash
ansible all -i inventory.ini -m ping
```

All three servers responded successfully with:

```text
"ping": "pong"
```

I also configured `ansible.cfg` so that I could run Ansible commands without repeatedly specifying the inventory file.

---

# 2. Jinja2 Templates

I started by creating a Jinja2 template for an Nginx virtual host.

Template:

```text
templates/nginx-vhost.conf.j2
```

The template contains variables such as:

```jinja2
{{ http_port }}
{{ app_name }}
{{ ansible_hostname }}
```

Example:

```jinja2
server {
    listen {{ http_port }};

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

I created a playbook called:

```text
template-demo.yml
```

and ran:

```bash
ansible-playbook template-demo.yml --syntax-check
```

The syntax check completed successfully.

Then I used:

```bash
ansible-playbook template-demo.yml --diff
```

The `--diff` option helped me see how the Jinja2 variables were replaced with actual values.

For example:

```text
{{ app_name }}
      ↓
terraweek-app
```

and:

```text
{{ ansible_hostname }}
      ↓
ip-172-31-26-110
```

This helped me understand how templates can generate configuration files dynamically.

---

# 3. Nginx Verification and Troubleshooting

After running the playbook, I checked the generated Nginx configuration:

```bash
sudo cat /etc/nginx/conf.d/terraweek-app.conf
```

The configuration was generated correctly.

I also tested the Nginx configuration:

```bash
sudo nginx -t
```

The result was:

```text
syntax is ok
test is successful
```

However, when I ran:

```bash
curl http://localhost
```

I still received the default Nginx welcome page.

This was a useful troubleshooting experience.

Instead of assuming that the successful Ansible run meant everything was working, I checked the actual Nginx configuration and found that the default Nginx site was still being served.

After removing the default site, testing the configuration again and restarting Nginx, the custom application page was displayed.

This taught me an important lesson:

> Automation doesn't eliminate troubleshooting — it makes troubleshooting more systematic.

---

# 4. Creating an Ansible Role

Next, I created a reusable Ansible role.

I generated the role structure using:

```bash
ansible-galaxy init roles/webserver
```

The structure became:

```text
roles/
└── webserver/
    ├── defaults/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── meta/
    │   └── main.yml
    ├── tasks/
    │   └── main.yml
    ├── templates/
    │   ├── nginx.conf.j2
    │   ├── vhost.conf.j2
    │   └── index.html.j2
    ├── vars/
    │   └── main.yml
    └── README.md
```

Instead of creating another separate README file for the role, I am documenting the complete project in this single Day 71 README.

---

# 5. Role Defaults

I defined default variables in:

```text
roles/webserver/defaults/main.yml
```

```yaml
---
http_port: 80
app_name: myapp
max_connections: 512
app_env: development
```

These are default values and can be overridden from the playbook.

For example:

```yaml
roles:
  - role: webserver
    vars:
      app_name: terraweek
      http_port: 80
      app_env: production
```

---

# 6. Role Tasks

The main tasks are defined in:

```text
roles/webserver/tasks/main.yml
```

The role performs the following tasks:

1. Install Nginx
2. Deploy the Nginx configuration
3. Deploy the virtual host configuration
4. Create the application web root
5. Deploy the HTML page
6. Start and enable Nginx

Because my target servers are Ubuntu, I used `apt` instead of `yum`.

Example:

```yaml
- name: Install Nginx
  apt:
    name: nginx
    state: present
    update_cache: true
```

---

# 7. Jinja2 Templates Inside the Role

I created three templates.

## nginx.conf.j2

This template uses:

```jinja2
{{ max_connections }}
```

to dynamically configure Nginx worker connections.

## vhost.conf.j2

This template uses:

```jinja2
{{ http_port }}
{{ app_name }}
{{ ansible_hostname }}
```

to create the Nginx virtual host.

## index.html.j2

The HTML page displays information about the server:

```html
<h1>{{ app_name }}</h1>

<p>Server: {{ ansible_hostname }}</p>

<p>IP: {{ ansible_default_ipv4.address }}</p>

<p>Environment: {{ app_env | default('development') }}</p>

<p>Managed by Ansible</p>
```

This allows the same role to be reused with different application names and environments.

---

# 8. Ansible Handler

I created a handler in:

```text
roles/webserver/handlers/main.yml
```

```yaml
---
- name: Restart Nginx
  service:
    name: nginx
    state: restarted
```

The Nginx configuration tasks use:

```yaml
notify: Restart Nginx
```

This means Nginx is restarted when the configuration changes.

---

# 9. Running the Custom Role

I created:

```text
site.yml
```

with:

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
        app_env: production
```

I first checked the syntax:

```bash
ansible-playbook site.yml --syntax-check
```

Then I ran:

```bash
ansible-playbook site.yml
```

After execution, I verified Nginx using:

```bash
sudo nginx -t
```

and:

```bash
curl http://localhost
```

The custom application page was displayed successfully.

---

# 10. Ansible Galaxy

After creating my own role, I practiced using community roles from Ansible Galaxy.

I searched for roles using:

```bash
ansible-galaxy search nginx --platforms EL
```

and:

```bash
ansible-galaxy search mysql
```

I also searched for Docker:

```bash
ansible-galaxy search docker
```

I installed the Docker role:

```bash
ansible-galaxy install geerlingguy.docker
```

Then I checked the installed roles:

```bash
ansible-galaxy list
```

---

# 11. Using a Galaxy Role

I created:

```text
docker-setup.yml
```

```yaml
---
- name: Install Docker using Galaxy role
  hosts: app
  become: true

  roles:
    - geerlingguy.docker
```

I ran:

```bash
ansible-playbook docker-setup.yml
```

After the playbook completed, I verified Docker on the app server:

```bash
docker --version
```

and:

```bash
sudo systemctl status docker
```

This showed me how a community role can provide reusable automation without writing every task from scratch.

---

# 12. requirements.yml

I also practiced managing roles using a requirements file.

I created:

```text
requirements.yml
```

```yaml
---
roles:
  - name: geerlingguy.docker
    version: "7.4.1"

  - name: geerlingguy.ntp
```

Then I installed the roles using:

```bash
ansible-galaxy install -r requirements.yml
```

Using `requirements.yml` makes it easier to manage multiple roles and specify versions.

---

# 13. Ansible Vault

The next part was protecting sensitive information.

I created the directory:

```bash
mkdir -p group_vars/db
```

Then I created an encrypted Vault file:

```bash
ansible-vault create group_vars/db/vault.yml
```

Inside the encrypted file, I stored database credentials and an API key.

Example:

```yaml
---
vault_db_password: MyDatabasePassword123
vault_db_root_password: MyRootPassword456
vault_api_key: my-example-api-key
```

I did not store these values in plain text in Git.

---

# 14. Verifying the Vault

When I run:

```bash
cat group_vars/db/vault.yml
```

I see encrypted content beginning with:

```text
$ANSIBLE_VAULT;1.1;AES256
```

To view the decrypted values:

```bash
ansible-vault view group_vars/db/vault.yml
```

To edit the encrypted file:

```bash
ansible-vault edit group_vars/db/vault.yml
```

I can also encrypt an existing file using:

```bash
ansible-vault encrypt secrets.yml
```

---

# 15. Vault Password File

For automation, I created:

```text
.vault_pass
```

and restricted its permissions:

```bash
chmod 600 .vault_pass
```

I also added it to `.gitignore`:

```text
.vault_pass
```

I can then run the playbook using:

```bash
ansible-playbook site.yml --vault-password-file .vault_pass
```

This is more suitable for automated execution than manually entering the Vault password every time.

The password file itself should never be committed to GitHub.

---

# 16. Using Vault Variables in a Template

I created:

```text
templates/db-config.j2
```

```jinja2
# Database Configuration -- Managed by Ansible

DB_HOST={{ ansible_default_ipv4.address }}

DB_PORT={{ db_port | default(3306) }}

DB_PASSWORD={{ vault_db_password }}

DB_ROOT_PASSWORD={{ vault_db_root_password }}
```

The sensitive values come from the encrypted Vault file.

---

# 17. Final site.yml

I combined the custom role, Galaxy role, Jinja2 template and Vault into the final playbook:

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
        app_env: production


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

I checked the syntax using:

```bash
ansible-playbook site.yml --syntax-check --vault-password-file .vault_pass
```

Then I ran:

```bash
ansible-playbook site.yml --vault-password-file .vault_pass
```

---

# 18. Final Verification

## Web Server

```bash
sudo nginx -t
```

```bash
curl http://localhost
```

## App Server

```bash
docker --version
```

## Database Server

```bash
sudo cat /etc/db-config.env
```

I also checked the permissions:

```bash
sudo stat -c "%a %n" /etc/db-config.env
```

Expected:

```text
600 /etc/db-config.env
```

The configuration file is therefore readable only by the appropriate privileged user.

---

# 19. Testing Idempotency

I ran the final playbook again:

```bash
ansible-playbook site.yml --vault-password-file .vault_pass
```

On the second run, already-configured resources should not be changed unnecessarily.

This helped me understand one of the important benefits of Ansible: **idempotent automation**.

---



# 20. Roles vs Playbooks vs Ad-Hoc Commands

### Ad-Hoc Commands

I use ad-hoc commands for quick tasks.

Example:

```bash
ansible all -m ping
```

### Playbooks

I use playbooks when I need to define a sequence of automation tasks.

Example:

```bash
ansible-playbook site.yml
```

### Roles

I use roles when automation becomes larger and needs to be organized into reusable components.

A role separates:

```text
Tasks
Handlers
Templates
Variables
Defaults
Metadata
```

This makes the automation easier to maintain and reuse.

---



