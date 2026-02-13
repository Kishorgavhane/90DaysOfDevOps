# Day 20 – Log Analyzer & Report Generator

# Task 1: Input and Validation
- `log_analyzer.sh`
- Generate log_report_<date>.txt
- Validate input properly
- Count errors
- Extract critical events
- Find top 5 errors
- Generate summary report

**Create `log_analyzer.sh`**
```bash
#!/bin/bash
set -euo pipefail

# Define log file path
log_file_path="sample_log.log"

# Define number of log lines
num_lines=100

# If file already exists, stop
if [ -e "$log_file_path" ]; then
    echo "Error: File already exists at $log_file_path."
    exit 1
fi

# List of possible log message levels
log_levels=("INFO" "DEBUG" "ERROR" "WARNING" "CRITICAL")

# List of possible error messages
error_messages=("Failed to connect" "Disk full" "Segmentation fault" "Invalid input" "Out of memory")

# Function to generate a random log line
generate_log_line() {
    local log_level="${log_levels[$((RANDOM % ${#log_levels[@]}))]}"
    local error_msg=""

    if [ "$log_level" == "ERROR" ]; then
        error_msg="${error_messages[$((RANDOM % ${#error_messages[@]}))]}"
    fi

    echo "$(date '+%Y-%m-%d %H:%M:%S') $log_level $error_msg"
}

# Create the log file
touch "$log_file_path"

for ((i=0; i<num_lines; i++)); do
    generate_log_line >> "$log_file_path"
done

echo "Log file created at: $log_file_path with $num_lines lines."
```

**Now i run script and cheack**
```bash
head sample_log.log
```
**Output:**

<img width="1151" height="234" alt="image" src="https://github.com/user-attachments/assets/6f5b1feb-9e99-4abb-8f47-621848d4249f" />


**I Learned:**
- How to accept arguments in shell script `($1)`
- How to validate argument count `($#)`
- How to check if a file exists `(-f)`
- How to safely exit script using `exit 1`
- Why set `-euo pipefail` is important

---

## Task 2: Error Count

```bash
#!/bin/bash
set -euo pipefail

# Task 1: Input Validation
if [ $# -ne 1 ]; then
    echo "Usage: $0 <log_file>"
    exit 1
fi

LOG_FILE="$1"

if [ ! -f "$LOG_FILE" ]; then
    echo "Error: Log file does not exist"
    exit 1
fi

# Task 2: Error Count
echo "---- Error Count ----"
ERROR_COUNT=$(grep -Eic "ERROR|Failed" "$LOG_FILE")
echo "Total Errors Found: $ERROR_COUNT"
```

**Now i run script and cheack**
```bash
./log_analyzer.sh sample_log.log
```
**output:**

<img width="1151" height="234" alt="image" src="https://github.com/user-attachments/assets/866f3b92-b288-4577-9323-3f7fb293fdab" />

**Why This Matters**
**In production:**
- ERROR means something broke.
- Failed means something did not succeed.
- Counting them gives quick health status.


**I Learned:**
- Using `grep -Eic`
- Using regex `(ERROR|Failed)`
- Case insensitive matching
- Counting matches directly

---

## Task 3 – Critical Events

### Why This Is Important
CRITICAL usually means:
- Disk almost full
- Database crash
- Major failure
These need immediate attention.

```bash
#!/bin/bash
set -euo pipefail

# ---------------------------
# Task 1: Input & Validation
# ---------------------------

if [ $# -ne 1 ]; then
    echo "Usage: $0 <log_file>"
    exit 1
fi

LOG_FILE="$1"

if [ ! -f "$LOG_FILE" ]; then
    echo "Error: Log file does not exist"
    exit 1
fi

# ---------------------------
# Task 2: Error Count
# ---------------------------

echo "---- Error Count ----"

ERROR_COUNT=$(grep -Eic "ERROR|Failed" "$LOG_FILE")

echo "Total Errors Found: $ERROR_COUNT"

# ---------------------------
# Task 3: Critical Events
# ---------------------------

echo ""
echo "--- Critical Events ---"

FOUND=false

while IFS=: read -r line content
do
    echo "Line $line: $content"
    FOUND=true
done < <(grep -n "CRITICAL" "$LOG_FILE" || true)

if [ "$FOUND" = false ]; then
    echo "No critical events found"
fi
```

**Output:**

<img width="1151" height="483" alt="image" src="https://github.com/user-attachments/assets/adf68bf2-6abd-416b-8dc8-e7d94b1a70b7" />


**I Learned:**
- `grep -n` for line numbers
- Capturing output in variables
- Handling no-match case with `|| true`
- Conditional output formatting

---

## Task 4: Top Error Messages

```bash
#!/bin/bash
set -euo pipefail

# ---------------------------
# Task 1: Input & Validation
# ---------------------------

if [ $# -ne 1 ]; then
    echo "Usage: $0 <log_file>"
    exit 1
fi

LOG_FILE="$1"

if [ ! -f "$LOG_FILE" ]; then
    echo "Error: Log file does not exist"
    exit 1
fi

# ---------------------------
# Task 2: Error Count
# ---------------------------

echo "---- Error Count ----"

ERROR_COUNT=$(grep -Eic "ERROR|Failed" "$LOG_FILE")

echo "Total Errors Found: $ERROR_COUNT"

# ---------------------------
# Task 3: Critical Events
# ---------------------------

echo ""
echo "--- Critical Events ---"

# ---------------------------
# Task 4: Top Error Messages
# ---------------------------

echo ""
echo "--- Top 5 Error Messages ---"

TOP_ERRORS=$(grep "ERROR" "$LOG_FILE" \
    | awk '{$1=$2=$3=""; sub(/^ +/, ""); print}' \
    | sort \
    | uniq -c \
    | sort -rn \
    | head -5)

if [ -z "$TOP_ERRORS" ]; then
    echo "No error messages found"
else
    echo "$TOP_ERRORS"
fi


FOUND=false

while IFS=: read -r line content
do
    echo "Line $line: $content"
    FOUND=true
done < <(grep -n "CRITICAL" "$LOG_FILE" || true)

if [ "$FOUND" = false ]; then
    echo "No critical events found"
fi
```

**Output:**

<img width="1151" height="609" alt="image" src="https://github.com/user-attachments/assets/8f4735ff-adf1-493a-bf31-adfa52d659ae" />


**I Learned:**
- Piping commands
- `awk` to remove fields
- `sort`
- `uniq -c`
- `sort -rn`
- `head -5`

---

## Task 5: Summary Report

```bash
#!/bin/bash
set -euo pipefail

# ---------------------------
# Task 1: Input & Validation
# ---------------------------

if [ $# -ne 1 ]; then
    echo "Usage: $0 <log_file>"
    exit 1
fi

LOG_FILE="$1"

if [ ! -f "$LOG_FILE" ]; then
    echo "Error: Log file does not exist"
    exit 1
fi

# ---------------------------
# Task 2: Error Count
# ---------------------------

echo "---- Error Count ----"

ERROR_COUNT=$(grep -Eic "ERROR|Failed" "$LOG_FILE")

echo "Total Errors Found: $ERROR_COUNT"

# ---------------------------
# Task 3: Critical Events
# ---------------------------

echo ""
echo "--- Critical Events ---"

CRITICAL_OUTPUT=$(grep -n "CRITICAL" "$LOG_FILE" || true)

if [ -z "$CRITICAL_OUTPUT" ]; then
    echo "No critical events found"
else
    echo "$CRITICAL_OUTPUT"
fi

# ---------------------------
# Task 4: Top Error Messages
# ---------------------------

echo ""
echo "--- Top 5 Error Messages ---"

TOP_ERRORS=$(grep "ERROR" "$LOG_FILE" \
    | awk '{$1=$2=$3=""; sub(/^ +/, ""); print}' \
    | sort \
    | uniq -c \
    | sort -rn \
    | head -5)

if [ -z "$TOP_ERRORS" ]; then
    echo "No error messages found"
else
    echo "$TOP_ERRORS"
fi

# ---------------------------
# Task 5: Summary Report
# ---------------------------

DATE=$(date +%Y-%m-%d)
REPORT_FILE="log_report_${DATE}.txt"
TOTAL_LINES=$(wc -l < "$LOG_FILE")

{
echo "================================="
echo "Log Analysis Report"
echo "Date of Analysis: $DATE"
echo "Log File Name: $LOG_FILE"
echo "Total Lines Processed: $TOTAL_LINES"
echo "Total Error Count: $ERROR_COUNT"
echo ""
echo "--- Top 5 Error Messages ---"
if [ -z "$TOP_ERRORS" ]; then
    echo "No error messages found"
else
    echo "$TOP_ERRORS"
fi
echo ""
echo "--- Critical Events ---"
if [ -z "$CRITICAL_OUTPUT" ]; then
    echo "No critical events found"
else
    echo "$CRITICAL_OUTPUT"
fi
echo "================================="
} > "$REPORT_FILE"

echo ""
echo "Summary report generated: $REPORT_FILE"
```

**Output:**

<img width="1151" height="634" alt="image" src="https://github.com/user-attachments/assets/1f1e7fa6-f1de-4683-8220-2287f2368501" />


<img width="1151" height="634" alt="image" src="https://github.com/user-attachments/assets/a3fe0155-b7eb-48b8-a826-d4ce7097d33c" />

**I Learned:**
- Creating dynamic file names using `date`
- Output redirection `>`
- Using `{ }` block for grouped output
- Writing structured report to file
- Using `wc -l` for total lines

---

## Task 6 : Archive Processed Logs

```bash
#!/bin/bash
set -euo pipefail

# ---------------------------
# Task 1: Input & Validation
# ---------------------------

if [ $# -ne 1 ]; then
    echo "Usage: $0 <log_file>"
    exit 1
fi

LOG_FILE="$1"

if [ ! -f "$LOG_FILE" ]; then
    echo "Error: Log file does not exist"
    exit 1
fi

# ---------------------------
# Task 2: Error Count
# ---------------------------

echo "---- Error Count ----"

ERROR_COUNT=$(grep -Eic "ERROR|Failed" "$LOG_FILE")

echo "Total Errors Found: $ERROR_COUNT"

# ---------------------------
# Task 3: Critical Events
# ---------------------------

echo ""
echo "--- Critical Events ---"

CRITICAL_OUTPUT=$(grep -n "CRITICAL" "$LOG_FILE" || true)

if [ -z "$CRITICAL_OUTPUT" ]; then
    echo "No critical events found"
else
    echo "$CRITICAL_OUTPUT"
fi

# ---------------------------
# Task 4: Top Error Messages
# ---------------------------

echo ""
echo "--- Top 5 Error Messages ---"

TOP_ERRORS=$(grep "ERROR" "$LOG_FILE" \
    | awk '{$1=$2=$3=""; sub(/^ +/, ""); print}' \
    | sort \
    | uniq -c \
    | sort -rn \
    | head -5)

if [ -z "$TOP_ERRORS" ]; then
    echo "No error messages found"
else
    echo "$TOP_ERRORS"
fi

# ---------------------------
# Task 5: Summary Report
# ---------------------------

DATE=$(date +%Y-%m-%d)
REPORT_FILE="log_report_${DATE}.txt"
TOTAL_LINES=$(wc -l < "$LOG_FILE")

{
echo "================================="
echo "Log Analysis Report"
echo "Date of Analysis: $DATE"
echo "Log File Name: $LOG_FILE"
echo "Total Lines Processed: $TOTAL_LINES"
echo "Total Error Count: $ERROR_COUNT"
echo ""
echo "--- Top 5 Error Messages ---"
if [ -z "$TOP_ERRORS" ]; then
    echo "No error messages found"
else
    echo "$TOP_ERRORS"
fi
echo ""
echo "--- Critical Events ---"
if [ -z "$CRITICAL_OUTPUT" ]; then
    echo "No critical events found"
else
    echo "$CRITICAL_OUTPUT"
fi
echo "================================="
} > "$REPORT_FILE"

echo ""
echo "Summary report generated: $REPORT_FILE"

# ---------------------------
# Task 6: Archive Processed Logs
# ---------------------------

ARCHIVE_DIR="archive"

mkdir -p "$ARCHIVE_DIR"

mv "$LOG_FILE" "$ARCHIVE_DIR/"

echo "Log file moved to $ARCHIVE_DIR/"
```

**Output:**


<img width="1151" height="157" alt="image" src="https://github.com/user-attachments/assets/c5467977-94fe-4582-9d0c-16b0f732200b" />

**I Learned:**
- Creating directory using `mkdir -p`
- Moving files using `mv`
- Keeping working directory clean
- Automation cleanup process

---


