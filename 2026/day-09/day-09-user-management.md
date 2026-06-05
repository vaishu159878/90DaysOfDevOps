# Day 09 – Linux User & Group Management Challenge

## Overview

Today I practiced Linux user and group management by performing real administration tasks on Ubuntu Linux.


# Users & Groups Created

## Users
- tokyo
- berlin
- professor
- nairobi

## Groups
- developers
- admins
- project-team

---

# Group Assignments

 User        Groups                     

 tokyo       developers, project-team   
 berlin      developers, admins         
 professor   admins                     
 nairobi     project-team               

---

# Directories Created

 Directory             Group Owner   Permissions 

 /opt/dev-project      developers    775         
 /opt/team-workspace   project-team  775         

---

# Commands Used


# Create Users
sudo useradd -m tokyo
sudo useradd -m berlin
sudo useradd -m professor
sudo useradd -m nairobi

# Set Passwords
sudo passwd tokyo
sudo passwd berlin
sudo passwd professor
sudo passwd nairobi

# Verify Users
cat /etc/passwd
ls /home

# Create Groups
sudo groupadd developers
sudo groupadd admins
sudo groupadd project-team

# Add Users to Groups
sudo usermod -aG developers tokyo
sudo usermod -aG developers,admins berlin
sudo usermod -aG admins professor
sudo usermod -aG project-team nairobi
sudo usermod -aG project-team tokyo

# Verify Groups
cat /etc/group
groups tokyo
groups berlin
groups professor
groups nairobi

# Create Shared Directories
sudo mkdir /opt/dev-project
sudo mkdir /opt/team-workspace

# Change Group Ownership
sudo chgrp developers /opt/dev-project
sudo chgrp project-team /opt/team-workspace

# Set Permissions
sudo chmod 775 /opt/dev-project
sudo chmod 775 /opt/team-workspace

# Test File Creation
sudo -u tokyo touch /opt/dev-project/tokyo-file
sudo -u berlin touch /opt/dev-project/berlin-file
sudo -u nairobi touch /opt/team-workspace/nairobi-file

# Verify Permissions
ls -ld /opt/dev-project
ls -ld /opt/team-workspace

# Verify Files
ls -l /opt/dev-project
ls -l /opt/team-workspace


# Screenshots

img1.png
img2.png

