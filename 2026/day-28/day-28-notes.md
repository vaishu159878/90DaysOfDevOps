# Day 28 – Revision Day Notes

## Self-Assessment Checklist

### Linux

| Topic                            | Status               |
| -------------------------------- | -------------------- |
| File system navigation           | ✅ Can do confidently |
| Process management               | ✅ Can do confidently |
| systemd services                 | ✅ Can do confidently |
| Text file editing                | ✅ Can do confidently |
| Troubleshooting CPU/Memory/Disk  | ✅ Can do confidently |
| Linux file system hierarchy      | ✅ Can do confidently |
| User & Group management          | ✅ Can do confidently |
| File permissions (chmod)         | ✅ Can do confidently |
| Ownership (chown/chgrp)          | ✅ Can do confidently |
| LVM management                   | ⚠️ Need to revisit   |
| Network troubleshooting commands | ⚠️ Need to revisit   |
| DNS, IP, Subnets, Ports          | ⚠️ Need to revisit   |

### Shell Scripting

| Topic                 | Status                |
| --------------------- | --------------------- |
| Variables & Arguments | ✅ Can do confidently  |
| Conditions            | ✅ Can do confidently  |
| Loops                 | ✅ Can do confidently  |
| Functions             | ✅ Can do confidently  |
| Text processing tools | ⚠️ Need more practice |
| Error handling        | ⚠️ Need more practice |
| Crontab scheduling    | ✅ Can do confidently  |

### Git & GitHub

| Topic                | Status                |
| -------------------- | --------------------- |
| Git basics           | ✅ Can do confidently  |
| Branching            | ✅ Can do confidently  |
| Push/Pull            | ✅ Can do confidently  |
| Clone vs Fork        | ✅ Can do confidently  |
| Merge                | ✅ Can do confidently  |
| Rebase               | ✅ Can do confidently  |
| Git Stash            | ✅ Can do confidently  |
| Cherry Pick          | ✅ Can do confidently  |
| Squash Merge         | ✅ Can do confidently  |
| Reset & Revert       | ✅ Can do confidently  |
| Branching Strategies | ⚠️ Need more revision |
| GitHub CLI           | ⚠️ Need more revision  |

---

## Topics Revisited Today

### 1. LVM (Logical Volume Manager)

### What I Re-learned

* Physical Volume (PV) is created from disks or partitions.
* Volume Group (VG) combines multiple PVs.
* Logical Volume (LV) is created from a VG.
* LVM allows resizing storage without repartitioning disks.
* Useful in production environments where storage requirements change frequently.

### Important Commands

```bash
pvcreate /dev/xvdb
vgcreate data_vg /dev/xvdb
lvcreate -L 5G -n app_lv data_vg

mkfs.ext4 /dev/data_vg/app_lv
mount /dev/data_vg/app_lv /mnt/data
```

---

### 2. Networking

### What I Re-learned

* DNS converts domain names into IP addresses.
* IP Address uniquely identifies a device.
* Ports identify services running on a system.
* Subnets divide large networks into smaller networks.

### Useful Commands

```bash
ping google.com
curl https://google.com
dig google.com
nslookup google.com
ss -tulpn
netstat -tulpn
```

Common Ports:

| Service | Port |
| ------- | ---- |
| SSH     | 22   |
| HTTP    | 80   |
| HTTPS   | 443  |
| DNS     | 53   |
| MySQL   | 3306 |

---

### 3. Shell Script Error Handling

### What I Re-learned

```bash
set -e
set -u
set -o pipefail
```

* `set -e` exits when a command fails.
* `set -u` treats undefined variables as errors.
* `set -o pipefail` catches failures in pipelines.

Example:

```bash
#!/bin/bash
set -euo pipefail

echo "Safe script execution"
```

---

# Quick Fire Answers

### 1. What does chmod 755 script.sh do?

Owner gets rwx permissions.
Group and others get r-x permissions.

---

### 2. Difference between process and service?

Process:

* Running program instance.

Service:

* Background process managed by systemd.

---

### 3. Find which process uses port 8080?

```bash
sudo ss -tulpn | grep 8080
```

or

```bash
sudo netstat -tulpn | grep 8080
```

---

### 4. What does set -euo pipefail do?

Makes shell scripts safer by stopping on errors, undefined variables, and failed pipeline commands.

---

### 5. Difference between git reset --hard and git revert?

`git reset --hard`

* Rewrites history.
* Removes commits locally.

`git revert`

* Creates a new commit that undoes changes.
* Safe for shared branches.

---

### 6. Recommended branching strategy for a team of 5 developers shipping weekly?

GitHub Flow or Trunk-Based Development.

Simple, fast, and suitable for frequent releases.

---

### 7. What does git stash do?

Temporarily saves uncommitted changes so you can switch branches without committing unfinished work.

```bash
git stash
git stash pop
```

---

### 8. Schedule a script daily at 3 AM?

```bash
crontab -e

0 3 * * * /home/ubuntu/backup.sh
```

---

### 9. Difference between git fetch and git pull?

`git fetch`

* Downloads changes only.

`git pull`

* Fetches and merges changes into the current branch.

---

### 10. What is LVM and why use it?

LVM provides flexible storage management.

Benefits:

* Resize volumes easily
* Combine multiple disks
* Easier storage administration

---

# Teach It Back

## Explaining Git Branching to a Beginner

Imagine multiple people writing different chapters of a book at the same time. Instead of editing the same document and causing conflicts, each person creates their own copy to work on. In Git, this copy is called a branch. Developers make changes in their branches without affecting the main project. Once the work is completed and reviewed, the branch is merged back into the main branch. Branching allows teams to develop features safely, collaborate efficiently, and maintain a stable version of the application.

---

## Revision Summary

Today I revised Linux, Networking, Shell Scripting, and Git/GitHub concepts from Day 1 to Day 27. I identified LVM, Networking, and advanced Shell Scripting as areas requiring additional practice. I reviewed key commands, concepts, and real-world use cases to strengthen my understanding before moving forward in the challenge.
