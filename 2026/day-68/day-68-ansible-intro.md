# Day 68 --- Introduction to Ansible and Inventory Setup

## 🚀 My Day 68 Ansible Journey

Today I started my Ansible journey.

While learning Terraform, I understood how infrastructure can be
provisioned using code. But after the servers are created, another
question comes up:

> **Who configures those servers and keeps them consistent?**

That is where Ansible comes in.

Instead of configuring every server manually, I wanted to understand how
I could manage multiple EC2 instances from a single control node.

For this lab, I focused on Ansible fundamentals, SSH connectivity,
inventory management, and ad-hoc commands.

------------------------------------------------------------------------

## 🏗️ Ansible Architecture

The basic architecture I worked with looks like this:

``` text
                         SSH
                          │
                          ▼
              ┌─────────────────────┐
              │    Control Node     │
              │                     │
              │      Ansible        │
              └──────────┬──────────┘
                         │
              ┌──────────┼──────────┐
              │          │          │
              ▼          ▼          ▼
          ┌───────┐  ┌───────┐  ┌───────┐
          │  Web  │  │  App  │  │  DB   │
          │Server │  │Server │  │Server │
          └───────┘  └───────┘  └───────┘
             Managed Nodes / EC2 Instances
```

### Control Node

The control node is the machine where I installed Ansible and from where
I ran my commands.

For this lab, I used an Ubuntu EC2 instance as my control node.

### Managed Nodes

Managed nodes are the remote servers that Ansible configures and
manages.

I planned my lab around:

-   Web server
-   App server
-   DB server

### Inventory

The inventory tells Ansible which servers it needs to manage and allows
me to organize them into groups.

### Modules

Modules are the units of work Ansible uses to perform tasks.

Some modules I practiced:

-   `ping`
-   `command`
-   `copy`
-   `apt`

### Playbooks

Playbooks are YAML files used to define repeatable automation.

I focused on ad-hoc commands today, and my next step is to start using
playbooks.

------------------------------------------------------------------------

## 🔐 Why Ansible Is Agentless

One of the concepts I found interesting today was that Ansible is
**agentless**.

I don't need to install an Ansible agent on every managed server.

For Linux servers, Ansible can communicate over SSH and execute the
required tasks remotely.

The basic flow is:

``` text
Control Node
     │
     │ SSH
     ▼
Managed EC2 Server
     │
     ▼
Execute Ansible Task
```

This keeps the setup simple and makes it easy to start managing new
servers.

------------------------------------------------------------------------

## 🖥️ Lab Environment

I used an Ubuntu EC2 instance as my Ansible control node.

The control node had:

``` text
OS:      Ubuntu
Ansible: 2.20.1
Python:  3.14.4
Path:    /usr/bin/ansible
```

I verified the installation with:

``` bash
ansible --version
```

The important output from my environment was:

``` text
ansible [core 2.20.1]
config file = None
executable location = /usr/bin/ansible
python version = 3.14.4
```

At this stage, `config file = None` was expected because I had not
created my project-level `ansible.cfg` yet.

------------------------------------------------------------------------

## 📋 Inventory Setup

I created an `inventory.ini` file to organize the managed servers.

For public sharing, I would redact the real IP addresses.

``` ini
[web]
web-server ansible_host=<WEB_PUBLIC_IP>

[app]
app-server ansible_host=<APP_PUBLIC_IP>

[db]
db-server ansible_host=<DB_PUBLIC_IP>

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/ansible.pem
```

The inventory gives each server a logical name and places it into a
group.

------------------------------------------------------------------------

## 🔎 Checking the Inventory

I used:

``` bash
ansible-inventory -i inventory.ini --graph
```

This helped me verify that Ansible understood my inventory structure and
groups.

------------------------------------------------------------------------

## 🟢 Testing Connectivity

The first Ansible command I wanted to get working was:

``` bash
ansible all -i inventory.ini -m ping
```

The expected result is:

``` text
web-server | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

app-server | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

db-server | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

This was an important milestone because it confirmed that the control
node could communicate with the managed servers.

> Screenshot: Add my `ansible all -m ping` screenshot here.

------------------------------------------------------------------------

## 🧪 Ad-Hoc Commands I Practiced

Ad-hoc commands are useful when I need to perform a quick task without
creating a complete playbook.

### 1. Check uptime

``` bash
ansible all -i inventory.ini -m command -a "uptime"
```

This lets me check the uptime of all managed servers from one terminal.

------------------------------------------------------------------------

### 2. Check memory

``` bash
ansible web -i inventory.ini -m command -a "free -h"
```

Here I targeted only the `web` group.

------------------------------------------------------------------------

### 3. Check disk space

``` bash
ansible all -i inventory.ini -m command -a "df -h"
```

This checks disk usage across all managed nodes.

------------------------------------------------------------------------

### 4. Install Git

Because I used Ubuntu, I used the `apt` module:

``` bash
ansible web -i inventory.ini -m apt -a "name=git state=present update_cache=yes" --become
```

I used `--become` because package installation requires elevated
privileges.

------------------------------------------------------------------------

### 5. Copy a file

First I created a small file:

``` bash
echo "Hello from Ansible" > hello.txt
```

Then I copied it to all managed servers:

``` bash
ansible all -i inventory.ini -m copy -a "src=hello.txt dest=/tmp/hello.txt"
```

I verified it with:

``` bash
ansible all -i inventory.ini -m command -a "cat /tmp/hello.txt"
```

Expected output:

``` text
Hello from Ansible
```

------------------------------------------------------------------------

## 🔑 Understanding `--become`

I learned that:

``` bash
--become
```

allows Ansible to execute a task with elevated privileges.

It is similar to using:

``` bash
sudo
```

manually on a Linux server.

For example, installing packages or managing system services generally
requires elevated privileges.

------------------------------------------------------------------------

## 🧩 Inventory Groups

I also practiced grouping hosts together.

``` ini
[application:children]
web
app

[all_servers:children]
application
db
```

This allows me to target multiple groups using one logical group.

For example:

``` bash
ansible application -i inventory.ini -m ping
```

This targets:

``` text
web + app
```

And:

``` bash
ansible all_servers -i inventory.ini -m ping
```

targets:

``` text
web + app + db
```

------------------------------------------------------------------------

## 🎯 Ansible Patterns

I practiced a few useful patterns.

### Web OR App

``` bash
ansible 'web:app' -i inventory.ini -m ping
```

### Everything except DB

``` bash
ansible 'all:!db' -i inventory.ini -m ping
```

These patterns make it easier to target exactly the servers I need.

------------------------------------------------------------------------

## ⚙️ Creating ansible.cfg

Initially, Ansible showed:

``` text
config file = None
```

So I created an `ansible.cfg` file in my project directory.

``` ini
[defaults]
inventory = inventory.ini
host_key_checking = False
remote_user = ubuntu
private_key_file = /home/ubuntu/ansible.pem
```

Now instead of writing:

``` bash
ansible all -i inventory.ini -m ping
```

I can use:

``` bash
ansible all -m ping
```

I can verify which configuration Ansible is using with:

``` bash
ansible --version
```

------------------------------------------------------------------------

## 🆚 `command` vs `shell`

One thing I wanted to understand clearly was the difference between the
`command` and `shell` modules.

### command

``` bash
ansible all -m command -a "uptime"
```

I use `command` for simple commands.

It does not process shell operators such as:

``` text
|
>
>>
&&
```

### shell

``` bash
ansible all -m shell -a "df -h | grep /dev"
```

The `shell` module runs the command through a shell, so shell features
such as pipes and redirection can be used.

My simple rule:

``` text
Simple command → command

Need shell features → shell
```

------------------------------------------------------------------------

