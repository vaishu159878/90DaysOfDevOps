# Day 07 – Linux File System Hierarchy & Scenario-Based Practice

## 📘 Objective

The goal of this task was to understand the Linux File System Hierarchy and practice real-world troubleshooting scenarios like a DevOps Engineer.

---

# 📂 Part 1 – Linux File System Hierarchy

## 1. `/` (Root Directory)

- The starting point of the Linux file system.
- All files and directories exist under this directory.

### Command Used

bash
ls -l /


### Observed Directories

- bin
- etc
- home
- var

### I would use this when...

I want to navigate and understand the Linux file system structure.

---

## 2. `/home`

- Stores personal files and directories of users.
- Each user gets their own home directory.

### Command Used

bash
ls -l /home


### Observed

- ubuntu user directory

### I would use this when...

I need to access user files and project directories.

---

## 3. `/root`

- Home directory of the root user.
- Accessible mainly by administrators.

### Command Used

bash
sudo ls -l /root


### Observed

- snap directory

### I would use this when...

Performing administrative tasks.

---

## 4. `/etc`

- Stores configuration files for the system and services.

### Command Used


ls -l /etc | head


### Example


cat /etc/hostname


### I would use this when...

Editing server and application configurations.

---

## 5. `/var/log`

- Contains system and service log files.
- Very important for troubleshooting.

### Command Used


ls -l /var/log | head


### Find Largest Log Files


du -sh /var/log/* 2>/dev/null | sort -h | tail -5


### I would use this when...

Investigating system or application issues.

---

## 6. `/tmp`

- Stores temporary files created by users and applications.

### Command Used


ls -l /tmp


### I would use this when...

Applications generate temporary runtime files.

---

## 7. `/bin`

- Contains essential Linux command binaries.

### Command Used


ls -l /bin | head


### Example


which ls


### I would use this when...

Finding essential Linux commands.

---

## 8. `/usr/bin`

- Contains user-level command binaries and applications.

### Command Used


ls -l /usr/bin | head


### Example


which python3


### I would use this when...

Locating installed applications and utilities.

---

## 9. `/opt`

- Used for optional and third-party applications.

### Command Used


ls -l /opt


### I would use this when...

Installing external applications.

---

# 🛠️ Part 2 – Scenario-Based Practice

## Scenario 1 – Service Not Starting

### Step 1: Check service status


systemctl status ssh


Why:
To verify whether the service is running or failed.

### Step 2: Check service logs


journalctl -u ssh -n 15


Why:
To identify issues from logs.

### Step 3: Check if service is enabled


systemctl is-enabled ssh


Why:
To verify whether the service starts automatically on boot.

### Step 4: List all services


systemctl list-units --type=service


Why:
To verify available services on the system.

---

## Scenario 2 – High CPU Usage

### Step 1: Monitor live CPU usage

top


Why:
Shows real-time CPU and memory utilization.

### Step 2: Find top CPU-consuming processes


ps aux --sort=-%cpu | head -10


Why:
Displays processes using the highest CPU.

---

## Scenario 3 – Finding Service Logs

### Step 1: Check Docker service status


systemctl status docker


Why:
Verifies whether Docker service exists and is running.

### Step 2: View SSH service logs


journalctl -u ssh -f

Why:
Monitors logs in real-time.

---

## Scenario 4 – File Permission Issue

### Step 1: Check current permissions


ls -l backup.sh


Why:
Verify whether execute permission exists.

### Step 2: Add execute permission


chmod +x backup.sh


Why:
Makes the script executable.

### Step 3: Run the script


./backup.sh


Why:
Verify the script executes successfully.

---

# 📚 What I Learned

- Linux file system hierarchy helps locate logs, binaries, configs, and user files.
- Logs are important for troubleshooting services.
- Permissions control access and execution of files.
- DevOps troubleshooting follows a step-by-step investigation process.

---

# 🚀 Commands Practiced

```bash
ls -l
cat
du -sh
sort
tail
systemctl
journalctl
top
ps aux
chmod +x
which
```

---

# 📸 Screenshots

img1.png
img2.png
img3.png
img4.png
img5.png


---



#90DaysOfDevOps #DevOpsKaJosh #TrainWithShubham