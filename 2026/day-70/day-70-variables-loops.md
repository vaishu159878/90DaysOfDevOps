# Day 70 -- Ansible Variables, Facts, Conditionals and Loops

## What I Practiced

Today I worked on making my Ansible playbooks more dynamic instead of
keeping everything static.

I practiced:

-   Variables
-   `group_vars`
-   `host_vars`
-   Ansible Facts
-   Conditionals with `when`
-   Loops
-   `register`
-   `debug`
-   Server health reporting

I worked with two Ubuntu servers:

-   `web-server`
-   `db-server`

------------------------------------------------------------------------

## Project Structure

``` text
ansible-practice/
├── inventory.ini
├── ansible.cfg
├── group_vars/
│   ├── all.yml
│   ├── web.yml
│   └── db.yml
├── host_vars/
│   └── web-server.yml
├── playbooks/
│   ├── variables-demo.yml
│   ├── site.yml
│   ├── facts-demo.yml
│   ├── conditional-demo.yml
│   ├── loops-demo.yml
│   └── server-report.yml
└── 
```

------------------------------------------------------------------------

# 1. Variables

I created `variables-demo.yml` and used variables instead of hardcoding
values.

Example:

``` yaml
vars:
  app_name: terraweek-app
  app_port: 8080
  app_dir: "/opt/{{ app_name }}"
  packages:
    - git
    - curl
    - wget
```

The playbook created:

``` text
/opt/terraweek-app
```

I also tested overriding variables from the command line:

``` bash
ansible-playbook playbooks/variables-demo.yml -e "app_name=my-custom-app app_port=9090"
```

The output changed to:

``` text
Deploying my-custom-app on port 9090 to /opt/my-custom-app
```

This helped me understand how extra variables can override playbook
values.

> Note: My managed servers are Ubuntu, so I used the `apt` module
> instead of `yum` for package installation.

------------------------------------------------------------------------

# 2. group_vars and host_vars

I moved variables outside the playbook so they can be managed
separately.

### `group_vars/all.yml`

``` yaml
ntp_server: pool.ntp.org
app_env: development

common_packages:
  - vim
  - htop
  - tree
```

### `group_vars/web.yml`

``` yaml
http_port: 80
max_connections: 1000

web_packages:
  - nginx
```

### `group_vars/db.yml`

``` yaml
db_port: 3306

db_packages:
  - mysql-server
```

### `host_vars/web-server.yml`

``` yaml
max_connections: 2000
custom_message: "This is the primary web server"
```

I tested variable precedence and saw that the host-specific value was
used for `web-server`.

I also tested an extra variable:

``` bash
ansible web-server -m debug -a "var=max_connections" -e "max_connections=5000"
```

The result was:

``` text
5000
```

------------------------------------------------------------------------

# 3. Ansible Facts

I used the `setup` module to inspect information about my servers.

Examples:

``` bash
ansible web-server -m setup -a "filter=ansible_os_family"
```

``` bash
ansible web-server -m setup -a "filter=ansible_distribution*"
```

``` bash
ansible web-server -m setup -a "filter=ansible_memtotal_mb"
```

``` bash
ansible web-server -m setup -a "filter=ansible_default_ipv4"
```

Some useful facts I practiced:

  Fact                             Why I would use it
  -------------------------------- ----------------------------
  `ansible_hostname`               Identify the server
  `ansible_distribution`           Check the operating system
  `ansible_distribution_version`   Check the OS version
  `ansible_memtotal_mb`            Check available memory
  `ansible_default_ipv4.address`   Get the primary IP address

My servers reported:

``` text
OS: Ubuntu 26.04
RAM: 908MB
```

------------------------------------------------------------------------

# 4. Conditionals

I created:

``` text
playbooks/conditional-demo.yml
```

I used `when` conditions so that tasks run only when they match the
server.

For example:

``` yaml
when: "'web' in group_names"
```

This allowed me to install Nginx only on the web server.

For the database server:

``` yaml
when: "'db' in group_names"
```

This allowed me to install MySQL only on the database server.

I also practiced conditions based on:

-   Operating system
-   Memory
-   Environment
-   Multiple conditions
-   OR conditions

The result showed that tasks were executed or skipped depending on the
host.

------------------------------------------------------------------------

# 5. Loops

I created:

``` text
playbooks/loops-demo.yml
```

I used loops to avoid writing the same task multiple times.

### Users

Created:

``` text
deploy
monitor
appuser
```

### Directories

Created:

``` text
/opt/app/logs
/opt/app/config
/opt/app/data
/opt/app/tmp
```

### Packages

Installed/verified:

``` text
git
curl
unzip
jq
```

Example:

``` yaml
loop: "{{ users }}"
```

I also practiced looping over directories and packages.

The playbook completed successfully on both servers.

------------------------------------------------------------------------

# 6. `loop` vs `with_items`

Older Ansible playbooks commonly used:

``` yaml
with_items:
  - git
  - curl
  - wget
```

I practiced the modern syntax:

``` yaml
loop:
  - git
  - curl
  - wget
```

`loop` makes the iteration syntax more consistent and is the recommended
approach for this exercise.

------------------------------------------------------------------------

# 7. Register and Server Health Report

I created:

``` text
playbooks/server-report.yml
```

The playbook checks:

-   Disk space
-   Memory
-   Running services
-   Operating system
-   IP address
-   RAM

I used `register` to store command results.

Example:

``` yaml
- name: Check disk space
  command: df -h /
  register: disk_result
```

The result can then be used later in the playbook.

------------------------------------------------------------------------

# 8. Server Health Report

The report was generated successfully on both servers.

### Web server

``` text
Server: web-server
OS: Ubuntu 26.04
IP: 172.31.26.110
RAM: 908MB
Disk: 17%
```

Report:

``` text
/tmp/server-report-web-server.txt
```

### Database server

``` text
Server: db-server
OS: Ubuntu 26.04
IP: 172.31.18.110
RAM: 908MB
Disk: 20%
```

Report:

``` text
/tmp/server-report-db-server.txt
```

I verified the files with:

``` bash
ansible all -m shell -a "ls -l /tmp/server-report-*"
```

and read the reports with:

``` bash
ansible all -m shell -a "cat /tmp/server-report-*"
```

------------------------------------------------------------------------

# What I Learned

This lab helped me understand that Ansible is not only about running the
same commands on multiple servers.

I can use:

``` text
Variables
   +
Facts
   +
Conditionals
   +
Loops
   +
Register
```

to make my automation more flexible.

For example, the same playbook can behave differently depending on
whether the host is a web server or a database server.

------------------------------------------------------------------------

# Key Takeaways

-   Variables make playbooks reusable.
-   `group_vars` help manage variables for groups of hosts.
-   `host_vars` allow host-specific configuration.
-   Facts provide information about managed servers.
-   `when` controls when a task should run.
-   Loops reduce repetitive tasks.
-   `register` stores task output for later use.
-   `debug` is useful while testing and troubleshooting.
-   Server reports can be generated automatically with Ansible.

