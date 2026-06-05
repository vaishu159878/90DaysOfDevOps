# Day 04 – Linux Practice: Processes and Services

## 📘 Objective

Today I practiced Linux process management, service monitoring, and troubleshooting commands on Ubuntu Linux.

The goal was to understand how to:
- Check running processes
- Inspect Linux services
- Monitor logs
- Practice basic troubleshooting steps

---

# 🖥️ Process Checks

## 1. View Running Processes


ps aux

Purpose:

Displays all running processes with detailed information.

## 2. Monitor System Processes in Real Time

top

Observed:

CPU usage
Memory usage
Active processes
System load

###⚙️ Service Checks

##3. Check SSH Service Status

systemctl status ssh

Observed:

SSH service was active and running.

### 4. List Running Services

systemctl list-units --type=service --state=running

Observed running services:

ssh.service
cron.service
systemd-journald.service
📄 Log Checks

### 5. View SSH Service Logs
journalctl -u ssh

Purpose:

Checked SSH login and service logs.

### 6. View Recent System Logs

tail -n 20 /var/log/syslog

Observed:

Recent system activity logs
Background service logs


### 🛠️ Mini Troubleshooting Practice

## Problem

Wanted to verify whether SSH service was running properly.

Steps Performed
Check service status
systemctl status ssh
Restart SSH service
sudo systemctl restart ssh
Verify service again
systemctl status ssh

## Result:

SSH service restarted successfully and was running properly.

### 📂 Additional Linux Practice

Practiced basic Linux commands:

mkdir
touch
cp
mv
ls

Created mini project folders and files for practice.

Installed and used the tree command:

sudo apt install tree
tree


## Screenshots

img1.png
img2.png
img3.png
img4.png
