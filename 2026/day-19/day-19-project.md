# Day 19 - Shell Scripting Project: Log Rotation, Backup & Crontab

## Task 1: Log Rotation Script

### Script: log_rotate.sh


#!/bin/bash

LOG_DIR=$1

if [ -z "$LOG_DIR" ]; then
    echo "Usage: $0 <log_directory>"
    exit 1
fi

if [ ! -d "$LOG_DIR" ]; then
    echo "Error: Directory does not exist."
    exit 1
fi

compressed=$(find "$LOG_DIR" -type f -name "*.log" -mtime +7 | wc -l)

find "$LOG_DIR" -type f -name "*.log" -mtime +7 -exec gzip {} \;

deleted=$(find "$LOG_DIR" -type f -name "*.gz" -mtime +30 | wc -l)

find "$LOG_DIR" -type f -name "*.gz" -mtime +30 -delete

echo "Compressed files: $compressed"
echo "Deleted files: $deleted"


### Sample Output


Compressed files: 2
Deleted files: 0


---

## Task 2: Server Backup Script

### Script: backup.sh


#!/bin/bash

SOURCE_DIR=$1
BACKUP_DIR=$2

if [ $# -ne 2 ]; then
    echo "Usage: $0 <source_directory> <backup_directory>"
    exit 1
fi

if [ ! -d "$SOURCE_DIR" ]; then
    echo "Error: Source directory does not exist."
    exit 1
fi

mkdir -p "$BACKUP_DIR"

DATE=$(date +%Y-%m-%d)
ARCHIVE_NAME="backup-$DATE.tar.gz"

tar -czf "$BACKUP_DIR/$ARCHIVE_NAME" "$SOURCE_DIR"

if [ $? -ne 0 ]; then
    echo "Backup creation failed."
    exit 1
fi

SIZE=$(du -h "$BACKUP_DIR/$ARCHIVE_NAME" | cut -f1)

echo "Backup Created Successfully"
echo "Archive: $ARCHIVE_NAME"
echo "Size: $SIZE"

find "$BACKUP_DIR" -type f -name "*.tar.gz" -mtime +14 -delete


### Sample Output


Backup Created Successfully
Archive: backup-2026-06-04.tar.gz
Size: 4.0K


---

## Task 3: Crontab

### Current Crontab


crontab -l


Output:


no crontab for ubuntu


### Cron Entries


# Run log rotation daily at 2 AM
0 2 * * * /home/ubuntu/scripts/log_rotate.sh /home/ubuntu/logs

# Run backup every Sunday at 3 AM
0 3 * * 0 /home/ubuntu/scripts/backup.sh /home/ubuntu/data /home/ubuntu/backups

# Run health check every 5 minutes
*/5 * * * * /home/ubuntu/scripts/health_check.sh

# Run maintenance daily at 1 AM
0 1 * * * /home/ubuntu/scripts/maintenance.sh


---

## Task 4: Scheduled Maintenance Script

### Script: maintenance.sh


#!/bin/bash

LOG_FILE="/home/ubuntu/maintenance.log"

log_message() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') : $1" >> "$LOG_FILE"
}

log_message "Maintenance Started"

./log_rotate.sh /home/ubuntu/logs >> "$LOG_FILE" 2>&1

./backup.sh /home/ubuntu/data /home/ubuntu/backups >> "$LOG_FILE" 2>&1

log_message "Maintenance Completed"


### Sample Output (maintenance.log)

2026-06-04 16:46:53 : Maintenance Started
Compressed files: 2
Deleted files: 0
Backup Created Successfully
Archive: backup-2026-06-04.tar.gz
Size: 4.0K
2026-06-04 16:46:53 : Maintenance Completed


---

## What I Learned

### 1. Automating Repetitive Tasks

I learned how shell scripts can automate routine system administration tasks such as log cleanup and backups.

### 2. Scheduling Jobs with Cron

I learned how to use cron jobs to run scripts automatically at specific times without manual intervention.

### 3. Importance of Logging and Error Handling

I learned that proper logging and error handling make scripts easier to monitor and troubleshoot.


## Screenshots

img1.png
img2.png
img3.png