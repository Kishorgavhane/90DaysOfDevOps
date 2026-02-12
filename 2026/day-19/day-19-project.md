# Day 19 – Real-World Shell Scripting Project

## Task 1: Log Rotation Script

## Servers generate logs daily:
- application logs
- system logs
- error logs

## If we don’t clean them:
- disk gets full
- server crashes

## So we:
- compress old logs
- delete very old logs

**Create a `log_rotate.sh`**
```bash
#!/bin/bash
set -euo pipefail

if [ $# -ne 1 ]; then
  echo "Usage: $0 <log_directory>"
  exit 1
fi

LOG_DIR="$1"

if [ ! -d "$LOG_DIR" ]; then
  echo "Error: Directory does not exist"
  exit 1
fi

echo "Compressing logs older than 7 days..."
COMPRESSED=$(find "$LOG_DIR" -name "*.log" -mtime +7 -type f -exec gzip {} \; -print | wc -l)

echo "Deleting .gz files older than 30 days..."
DELETED=$(find "$LOG_DIR" -name "*.gz" -mtime +30 -type f -delete -print | wc -l)

echo "Compressed files: $COMPRESSED"
echo "Deleted files: $DELETED"
```

**Run Like**
```text
./log_rotate.sh /var/log
```

**Output:**
```text
Compressing logs older than 7 days...
Deleting .gz files older than 30 days...
Compressed files: 0
Deleted files: 0
```

<img width="1245" height="163" alt="image" src="https://github.com/user-attachments/assets/78dbe890-6eb9-4ba9-9511-0b932ad9ea32" />


**What this does**
- Takes directory as argument
- Compresses `.log` older than 7 days
- Deletes `.gz` older than 30 days
- Prints count

---

## Task 2: Log Backup Script

**Why need?**
- regular backups
- timestamped archives
- auto cleanup

**Create `backup.sh`**
```
#!/bin/bash
set -euo pipefail

if [ $# -ne 2 ]; then
  echo "Usage: $0 <source_dir> <backup_destination>"
  exit 1
fi

SOURCE="$1"
DEST="$2"

if [ ! -d "$SOURCE" ]; then
  echo "Error: Source directory does not exist"
  exit 1
fi

TIMESTAMP=$(date +%Y-%m-%d)
ARCHIVE="$DEST/backup-$TIMESTAMP.tar.gz"

mkdir -p "$DEST"

tar -czf "$ARCHIVE" "$SOURCE"

if [ -f "$ARCHIVE" ]; then
  echo "Backup created: $ARCHIVE"
  ls -lh "$ARCHIVE"
else
  echo "Backup failed"
  exit 1
fi

# Delete backups older than 14 days
find "$DEST" -name "backup-*.tar.gz" -mtime +14 -delete

echo "Old backups cleaned"
```
**Run like**
```text
./backup.sh /home/ubuntu/data /home/ubuntu/backups
```
**Output:**

<img width="1245" height="246" alt="image" src="https://github.com/user-attachments/assets/7ab4759a-c1b1-4512-99f3-2d2562bf0f28" />


---

## Crontab Basics

- It runs scripts automatically at specific times.
- Instead of running scripts manually every day, we let the system handle it.
**Example:**
- Backup every Sunday automatically
- Rotate logs daily
- Run health checks every 5 minutes

## Cron Syntax Explained

```text
* * * * * command
| | | | |
| | | | └── Day of week (0-7) → 0 or 7 = Sunday
| | | └──── Month (1-12)
| | └────── Day of month (1-31)
| └──────── Hour (0-23)
└────────── Minute (0-59)
```

**Example:**
```text
0 2 * * * script.sh
```

**Meaning:**
- 0 minute
- 2 hour (2 AM)
- every day
- every month
- every day of week

## Common Time Examples
| Time                 | Cron Format   |
| -------------------- | ------------- |
| Every day at 2 AM    | `0 2 * * *`   |
| Every Sunday at 3 AM | `0 3 * * 0`   |
| Every 5 minutes      | `*/5 * * * *` |

## Crontab Entries (Do not apply without verification)

## Run log_rotate.sh every day at 2 AM

`0 2 * * * /home/ubuntu/shell_scripts/log_rotate.sh /var/log >> /var/log/log_rotate.log 2>&1`


## Run backup.sh every Sunday at 3 AM

`0 3 * * 0 /home/ubuntu/shell_scripts/backup.sh /home/ubuntu/data /home/ubuntu/backups >> /var/log/backup.log 2>&1`


## Run health_check.sh every 5 minutes

`*/5 * * * * /home/ubuntu/shell_scripts/health_check.sh >> /var/log/health_check.log 2>&1`

**Why We Added `>> logfile 2>&1`**
**Means:**
- Save normal output
- Save error output
- Append to log file

# Without logging, cron runs silently.

# How to Check Existing Cron Jobs
**Run:**
```bash
crontab -l
```

**To edit cron:**
```bash
crontab -e
```

---

## Combine Everything

**Create a `maintenance.sh`**

```bash
#!/bin/bash
set -euo pipefail

LOG_FILE="/var/log/maintenance.log"

echo "$(date): Starting maintenance" >> $LOG_FILE

./log_rotate.sh /var/log >> $LOG_FILE 2>&1
./backup.sh /home/ubuntu/data /home/ubuntu/backups >> $LOG_FILE 2>&1

echo "$(date): Maintenance completed" >> $LOG_FILE
```
**Cron entry (daily 1 AM):**
```bash
0 1 * * * /path/maintenance.sh
```

---

- **Today I worked on a real shell scripting project with log rotation, backup, and cron scheduling.**
- **I learned how to automate maintenance tasks and run them at specific times using crontab.**
- **I understood how to verify whether a cron job actually runs by checking logs and system status.**
- **This practice helped me move from basic scripting to more practical, production-style automation.**

