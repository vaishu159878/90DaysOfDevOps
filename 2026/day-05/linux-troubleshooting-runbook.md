# Day 05 – Linux Troubleshooting Drill

## Objective
Today I practiced Linux troubleshooting on an AWS EC2 Ubuntu server by checking CPU, memory, disk, network, and SSH logs.

## Commands Used & Observations

### uname -a

uname -a
Checked Linux kernel version and system architecture.

## lsb_release -a

lsb_release -a
Verified Ubuntu version details.

## mkdir /tmp/runbook-demo

mkdir /tmp/runbook-demo
Created temporary troubleshooting folder.

## cp /etc/hosts /tmp/runbook-demo/hosts-copy && ls -l /tmp/runbook-demo

cp /etc/hosts /tmp/runbook-demo/hosts-copy && ls -l /tmp/runbook-demo
Copied hosts file and verified permissions.

## top

top
Monitored running processes, CPU, and memory usage.

## free -h

free -h
Checked available RAM and swap memory.

## df -h

df -h
Verified available disk space.

## du -sh /var/log

du -sh /var/log
Checked log directory size.

## ss -tulpn

ss -tulpn
Viewed active listening ports and services.
 
## journalctl -u ssh

journalctl -u ssh
Reviewed SSH service logs and login activity.

## Quick Findings

CPU and memory usage were normal
Disk space was available
SSH service was active and running
Network ports were listening correctly
No major errors found in logs
If This Worsens
Restart SSH Service
sudo systemctl restart ssh
Monitor Live Logs
journalctl -u ssh -f
Check Firewall Status


## Screenshots

img1.png
img2.png
img3.png
img4.png
img5.png
