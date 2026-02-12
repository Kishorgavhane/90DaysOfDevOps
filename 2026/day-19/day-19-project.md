### Day 19 – Real-World Shell Scripting Project

## Task 1: Log Rotation Script

# Servers generate logs daily:
- application logs
- system logs
- error logs

# If we don’t clean them:
- disk gets full
- server crashes

# So we:
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

## Task 1: Log Backup Script

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

