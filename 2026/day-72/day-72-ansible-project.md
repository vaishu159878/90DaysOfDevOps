# Day 72 -- Ansible Project: Docker + Nginx Deployment

## Introduction

Today I worked on a complete Ansible project where I combined the
concepts I learned over the previous days into one practical deployment.

The goal was to start with an Ubuntu EC2 server and automate the setup
using Ansible.

The final setup was:

``` text
Ansible Control Node
        |
        | SSH
        v
   Ubuntu EC2
        |
        +---- Nginx :80
        |       |
        |       | Reverse Proxy
        |       v
        +---- Docker :8080
                |
                v
          nginx:latest
```

Instead of installing and configuring everything manually, I wanted
Ansible to handle the complete setup.

------------------------------------------------------------------------

## What I Built

I created an Ansible project with three custom roles:

-   `common` -- basic server configuration
-   `docker` -- Docker installation and container deployment
-   `nginx` -- Nginx installation and reverse proxy configuration

I also used:

-   Ansible inventory
-   Playbooks
-   Variables
-   Jinja2 templates
-   Handlers
-   Tags
-   Ansible Vault
-   Ansible collections
-   Idempotency testing

------------------------------------------------------------------------

## Project Structure

``` text
ansible-docker-project/
├── ansible.cfg
├── inventory.ini
├── site.yml
├── group_vars/
│   ├── all.yml
│   └── web/
│       ├── vars.yml
│       └── vault.yml
├── roles/
│   ├── common/
│   │   ├── defaults/
│   │   ├── handlers/
│   │   └── tasks/
│   │       └── main.yml
│   │
│   ├── docker/
│   │   ├── defaults/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   └── tasks/
│   │       └── main.yml
│   │
│   └── nginx/
│       ├── defaults/
│       │   └── main.yml
│       ├── handlers/
│       │   └── main.yml
│       ├── tasks/
│       │   └── main.yml
│       └── templates/
│           └── app-proxy.conf.j2
└── .gitignore
```

------------------------------------------------------------------------

## 1. Common Role

First, I created a common role for basic server configuration.

It handles:

-   Updating the apt cache
-   Installing common packages
-   Setting the hostname
-   Setting the timezone
-   Creating the `deploy` user

For Ubuntu, I used `apt` instead of `yum`.

Example:

``` yaml
- name: Install common packages
  apt:
    name: "{{ common_packages }}"
    state: present
```

I configured the timezone as:

``` yaml
timezone: Asia/Kolkata
```

------------------------------------------------------------------------

## 2. Docker Role

Next, I created the Docker role.

The role handles:

1.  Installing Docker dependencies
2.  Adding the Docker GPG key
3.  Adding the Docker repository
4.  Installing Docker
5.  Starting and enabling Docker
6.  Adding the required user to the Docker group
7.  Pulling the application image
8.  Running the application container
9.  Checking whether the application is responding

The container configuration was:

``` text
Image: nginx:latest
Container name: myapp
Host port: 8080
Container port: 80
```

The final port mapping was:

``` text
0.0.0.0:8080 -> 80/tcp
```

------------------------------------------------------------------------

## 3. Docker Troubleshooting

This was one of the useful parts of the project because the deployment
did not work perfectly on the first attempt.

My Ubuntu instance was running:

``` text
Ubuntu 26.04 LTS
Codename: resolute
```

During the Ansible check run, Docker installation failed with:

``` text
No package matching 'docker-ce' is available
```

I checked the Ubuntu version and the Docker package availability instead
of immediately installing Docker manually.

I found that `docker-ce` was not available from the repository
configuration being used.

I then adjusted the Docker repository configuration and continued the
deployment through Ansible.

This helped me understand that troubleshooting is an important part of
automation work.

------------------------------------------------------------------------

## 4. Docker Permission Issue

After Docker was installed, I initially got:

``` text
permission denied while trying to connect to the Docker API
```

when running:

``` bash
docker ps
```

The Docker service itself was working.

The problem was user permissions for the Docker socket.

I added the SSH user to the Docker group and reconnected to the server.

After logging in again:

``` bash
docker ps
```

worked successfully.

This reminded me that installing a service and being able to use it as a
normal user are two different things.

------------------------------------------------------------------------

## 5. Verify Docker Container

After the Docker role completed, I checked:

``` bash
docker --version
```

Output:

``` text
Docker version 29.8.1
```

Then:

``` bash
docker ps
```

The container was running:

``` text
CONTAINER ID   IMAGE          STATUS         PORTS
318491c33512   nginx:latest   Up             0.0.0.0:8080->80/tcp
```

The container name was:

``` text
myapp
```

------------------------------------------------------------------------

## 6. Test the Application

I tested the application directly through Docker:

``` bash
curl http://localhost:8080
```

The Nginx welcome page was returned successfully.

This confirmed that:

``` text
Host :8080
    ↓
Docker Container :80
    ↓
Nginx
```

was working.

------------------------------------------------------------------------

## 7. Nginx Role

After confirming Docker was working, I configured Nginx.

The Nginx role handles:

-   Installing Nginx
-   Removing the default site
-   Deploying the reverse proxy configuration
-   Enabling the application site
-   Testing the Nginx configuration
-   Starting and enabling Nginx
-   Reloading Nginx when configuration changes

The reverse proxy configuration uses a Jinja2 template.

The important part is:

``` nginx
upstream docker_app {
    server 127.0.0.1:8080;
}

server {
    listen 80;

    location / {
        proxy_pass http://docker_app;
    }
}
```

So requests coming to port 80 are forwarded to the Docker application on
port 8080.

------------------------------------------------------------------------

## 8. Nginx Configuration Test

Before relying on the configuration, I tested it with:

``` bash
sudo nginx -t
```

The result was:

``` text
syntax is ok
test is successful
```

This confirmed that the Nginx configuration was valid.

------------------------------------------------------------------------

## 9. Health Check

I also added a health endpoint:

``` text
/health
```

Testing:

``` bash
curl http://localhost/health
```

returned:

``` text
OK
```

------------------------------------------------------------------------

## 10. Ansible Vault

I used Ansible Vault to protect Docker Hub credentials.

The Vault file contains variables such as:

``` yaml
vault_docker_username: your-dockerhub-username
vault_docker_password: your-dockerhub-token
```

The file is encrypted instead of storing the credentials as plain text.

I also created a `.vault_pass` file for the lab and added it to
`.gitignore`.

I learned that secrets should not be committed directly to GitHub.

------------------------------------------------------------------------

## 11. Master Playbook

The complete deployment is controlled by:

``` text
site.yml
```

It runs the roles in this order:

``` text
common
   ↓
docker
   ↓
nginx
```

The command to deploy the complete setup is:

``` bash
ansible-playbook site.yml
```

So instead of manually repeating many installation and configuration
commands, I can use one playbook to bring the server to the required
state.

------------------------------------------------------------------------

## 12. Ansible Tags

I also used tags so that I can run only the part I need.

For Docker:

``` bash
ansible-playbook site.yml --tags docker
```

For Nginx:

``` bash
ansible-playbook site.yml --tags nginx
```

For common configuration:

``` bash
ansible-playbook site.yml --tags common
```

This is useful when I only want to change one part of the environment.

------------------------------------------------------------------------

## 13. Idempotency

One of the most important things I tested was idempotency.

I ran:

``` bash
ansible-playbook site.yml
```

Then I ran the same command again.

On the second run, Ansible reported mostly:

``` text
ok
```

with no unnecessary changes.

The final recap showed:

``` text
unreachable=0
failed=0
```

This helped me understand why idempotency is important in configuration
management.

I don't want my automation to make unnecessary changes every time it
runs.

------------------------------------------------------------------------

## 14. Final Verification

I verified the setup using:

``` bash
docker ps
```

``` bash
curl http://localhost:8080
```

``` bash
curl http://localhost
```

``` bash
curl http://localhost/health
```

``` bash
sudo nginx -t
```

The final flow was:

``` text
Browser
   |
   | HTTP :80
   v
Nginx Reverse Proxy
   |
   | 127.0.0.1:8080
   v
Docker Container
   |
   | :80
   v
Nginx Application
```

------------------------------------------------------------------------

## What I Learned

This project helped me connect many Ansible concepts together instead of
learning them separately.

### Concepts I used

  Concept       How I used it
  ------------- ----------------------------------------------------
  Inventory     Connected Ansible to the Ubuntu server
  Modules       `apt`, `user`, `service`, `template`, `file`, etc.
  Variables     Stored reusable configuration values
  Roles         Separated common, Docker and Nginx tasks
  Templates     Created dynamic Nginx configuration
  Handlers      Reloaded Nginx after configuration changes
  Tags          Ran selected parts of the deployment
  Vault         Protected Docker credentials
  Collections   Used `community.docker`
  Idempotency   Verified repeated playbook execution

------------------------------------------------------------------------

## Challenges I Faced

The biggest challenges were:

1.  Docker CE was not available with the initial repository
    configuration.
2.  Docker was installed but the SSH user initially did not have
    permission to access the Docker socket.
3.  I had to understand the difference between running a playbook in
    `--check` mode and actually applying changes.
4.  I verified each layer separately instead of assuming that a
    successful Ansible task meant the application was working.

These issues made the project more practical because I had to
troubleshoot instead of only following commands.

------------------------------------------------------------------------

## Final Result

I successfully automated a complete Docker + Nginx deployment on Ubuntu
using Ansible.

The final environment included:

``` text
Ubuntu EC2
   ↓
Ansible
   ↓
Docker
   ↓
Nginx Container
   ↓
Nginx Reverse Proxy
   ↓
Port 80
```

The biggest takeaway for me was:

> Automation is not only about writing YAML. It is also about
> understanding the system, troubleshooting failures, and making the
> setup repeatable.

