# Shell Scripting Cheat Sheet – Reference Guide

> Built as part of #90DaysOfDevOps  

---

# 🚀 Shell Scripting?

Shell scripting is writing a sequence of Linux commands inside a file to automate tasks.

Instead of typing commands manually, we create scripts to:
- Automate repetitive tasks
- Manage servers
- Monitor systems
- Process logs
- Deploy applications

---

# Quick Reference Table

| Topic | Key Syntax | Example |
|-------|------------|----------|
| Variable | VAR="value" | NAME="DevOps" |
| Argument | $1, $2 | ./script.sh arg1 |
| If | if [ condition ]; then | if [ -f file ]; then |
| For loop | for i in list; do | for i in 1 2 3; do |
| Function | name() { } | greet() { echo "Hi"; } |
| Grep | grep pattern file | grep -i "error" log.txt |
| Awk | awk '{print $1}' file | awk -F: '{print $1}' /etc/passwd |
| Sed | sed 's/old/new/g' file | sed -i 's/foo/bar/g' file |

---

# Task 1 – Basics

## 1. Shebang
```bash
#!/bin/bash
```
**Purpose:**
- Tells Linux to execute the script using Bash.
**Why it matters:**
- Ensures correct interpreter. Required for portability and cron jobs.
**Alternative:**
  
```bash
#!/usr/bin/env bash
```
## 2. Running a Script

**Make executable:**
```bash
chmod +x script.sh
```
**Run:**
```bash
./script.sh
```
**Run without execute permission:**
```bash
bash script.sh
```
- **`./` is required because current directory is not in PATH.**

## 3. Comments

**Single-line:**
```bash
# This is a comment
```
**Inline:**
```bash
echo "Hello"  # Prints greeting
```
## 4. Variables

**Declare:**
```bash
NAME="DevOps"
COUNT=10
```
**Use:**
```bash
echo $NAME
echo "$NAME"
```
- **⚠ No spaces around `=`**

### Quoting Rules

```bash
"$VAR"   # Expands variable safely (recommended)
'$VAR'   # Literal string, no expansion
```
**Always prefer:**
```bash
"$VAR"
```

## 5. Read User Input

**Basic:**
```bash
read NAME
```
**With prompt:**
```bash
read -p "Enter name: " NAME
```
**Silent input:**
```bash
read -s PASSWORD
```

## 6. Command-Line Arguments

If script runs as:
```bash
./script.sh file.txt
```

| Variable | Meaning                     |
| -------- | --------------------------- |
| `$0`     | Script name                 |
| `$1`     | First argument              |
| `$2`     | Second argument             |
| `$#`     | Total arguments             |
| `$@`     | All arguments               |
| `$?`     | Exit status of last command |

**Example:**
```bash
echo "Script: $0"
echo "File: $1"
echo "Total args: $#"
```
- 0 = success
- Non-zero = failure

```bash
#!/bin/bash

if [ $# -ne 1 ]; then
  echo "Usage: $0 <file>"
  exit 1
fi

FILE="$1"

echo "Processing $FILE"
```

---

# Task 2 – Operators & Conditionals

## 1. String Comparisons

- Used for comparing text values.
```bash
[ "$a" = "$b" ]    # Equal
[ "$a" != "$b" ]   # Not equal
[ -z "$var" ]      # True if empty
[ -n "$var" ]      # True if not empty
```
**Example:**
```bash
NAME="DevOps"

if [ "$NAME" = "DevOps" ]; then
  echo "Match"
fi
```
- **Always use quotes:**
```bash
[ "$var" = "value" ]
```

## 2. Integer Comparisons

- Used for numbers.
```bash
[ "$a" -eq 5 ]   # Equal
[ "$a" -ne 5 ]   # Not equal
[ "$a" -lt 5 ]   # Less than
[ "$a" -gt 5 ]   # Greater than
[ "$a" -le 5 ]   # Less than or equal
[ "$a" -ge 5 ]   # Greater than or equal
```
**Example:**
```bash
COUNT=10

if [ "$COUNT" -gt 5 ]; then
  echo "Greater than 5"
fi
```

- **Remember:**
- `=` is for strings
- `-eq` is for numbers

## 3. File Test Operators

- Used to check file or directory properties.
```bash
-f file   # Is regular file
-d dir    # Is directory
-e file   # Exists
-r file   # Readable
-w file   # Writable
-x file   # Executable
-s file   # Not empty (size > 0)
```

**Example:**
```bash
if [ -f "config.txt" ]; then
  echo "File exists"
fi
```

## 4. if / elif / else Syntax

- structure:
```bash
if [ condition ]; then
  commands
elif [ condition ]; then
  commands
else
  commands
fi
```
**Example:**
```bash
NUM=5

if [ "$NUM" -gt 10 ]; then
  echo "Greater"
elif [ "$NUM" -eq 10 ]; then
  echo "Equal"
else
  echo "Smaller"
fi
```
- **Every if block must end with:**
```bash
fi
```

## 5. Logical Operators

- Combine conditions.
**AND (&&)**
```bash
[ "$a" -gt 5 ] && echo "True"
```
**Meaning:** Run second command only if first succeeds.

**OR (||)**
```bash
[ -f file.txt ] || echo "File missing"
```
- Run second command if first fails.

**NOT (!)**
```bash
if [ ! -f file.txt ]; then
  echo "File not found"
fi
```
**Combined Example:**
```bash
if [ "$a" -gt 5 ] && [ "$a" -lt 20 ]; then
  echo "Between 5 and 20"
fi
```

## 6. Case Statements

- Used for multi-value matching.
- Cleaner than multiple if blocks.
**Syntax:**
```bash
case $var in
  pattern1)
    commands
    ;;
  pattern2)
    commands
    ;;
  *)
    default
    ;;
esac
```
**Example:**
```bash
ACTION="start"

case $ACTION in
  start)
    echo "Starting service"
    ;;
  stop)
    echo "Stopping service"
    ;;
  restart)
    echo "Restarting service"
    ;;
  *)
    echo "Invalid option"
    ;;
esac
```

```bash
case "$1" in
  deploy)
    echo "Deploying application"
    ;;
  rollback)
    echo "Rolling back"
    ;;
  *)
    echo "Usage: $0 {deploy|rollback}"
    ;;
esac
```
## Revision Table

| Type    | Operator | Meaning     |   |    |
| ------- | -------- | ----------- | - | -- |
| String  | =        | Equal       |   |    |
| String  | !=       | Not equal   |   |    |
| String  | -z       | Empty       |   |    |
| String  | -n       | Not empty   |   |    |
| Number  | -eq      | Equal       |   |    |
| Number  | -gt      | Greater     |   |    |
| File    | -f       | File exists |   |    |
| File    | -d       | Directory   |   |    |
| Logical | &&       | AND         |   |    |
| Logical |          |             |   | OR |
| Logical | !        | NOT         |   |    |


---

# Task 3 – Loops

## 1. for Loop
- Used when you know how many times to repeat or have a list.
**List-Based for Loop**
```bash
for item in one two three; do
  echo "$item"
done
```
**Example:**
```bash
for server in web1 web2 web3; do
  echo "Checking $server"
done
```
**Range-Based Loop**
```bash
for i in {1..5}; do
  echo "$i"
done
```

**C-Style for Loop**
```bash
for ((i=0; i<5; i++)); do
  echo "$i"
done
```

## 2. while Loop

- Runs while condition is true.
```bash
while [ condition ]; do
  commands
done
```
**Example:**
```bash
COUNT=1

while [ "$COUNT" -le 5 ]; do
  echo "$COUNT"
  COUNT=$((COUNT + 1))
done
```
```bash
while ! systemctl is-active --quiet nginx; do
  echo "Waiting for nginx..."
  sleep 2
done
```

## 3. until Loop

- Runs until condition becomes true.
- Opposite of `while`.
```bash
until [ condition ]; do
  commands
done
```
**Example:**
```bash
COUNT=1

until [ "$COUNT" -gt 5 ]; do
  echo "$COUNT"
  COUNT=$((COUNT + 1))
done
```

## 4. Loop Control

- **`break`**
Stops loop immediately.
```bash
for i in 1 2 3 4; do
  if [ "$i" -eq 3 ]; then
    break
  fi
  echo "$i"
done
```

- **`continue`**
Skips current iteration.
```bash
for i in 1 2 3 4; do
  if [ "$i" -eq 3 ]; then
    continue
  fi
  echo "$i"
done
```

## 5. Looping Over Files

Very common in DevOps.
```bash
for file in *.log; do
  echo "Processing $file"
done
```

**Example:**
```bash
for file in *.log; do
  rm "$file"
done
```
## 6. Looping Over Command Output

Used for processing dynamic results.
- **Using `while read`**
```bash
while read line; do
  echo "Line: $line"
done < file.txt
```
- **Loop Over Command Output**
```bash
ps aux | while read line; do
  echo "$line"
done
```
- **Safer Version**
```bash
while IFS= read -r line; do
  echo "$line"
done < file.txt
```

## Comparison
| Loop     | Use When                   |
| -------- | -------------------------- |
| for      | Fixed list or range        |
| while    | Condition-based repetition |
| until    | Run until condition true   |
| break    | Stop loop                  |
| continue | Skip iteration             |


---

# Task 4 – Functions

- **Note: A function is a reusable block of code. Instead of repeating commands, you define them once and call them when needed.**

## 1. Basic Function Syntax

```bash
function_name() {
  commands
}
```

**Or:**

```bash
function function_name {
  commands
}
```
- Both work.

## 2. Calling a Function

```bash
greet() {
  echo "Hello DevOps"
}

greet
```

## 3. Function with Arguments
- Functions can accept arguments like scripts.
```bash
add() {
  echo $(($1 + $2))
}

add 5 3
```
**Inside Function**
| Variable | Meaning         |
| -------- | --------------- |
| $1       | First argument  |
| $2       | Second argument |
| $@       | All arguments   |
| $#       | Total arguments |

## 4. Returning Values
```bash
check_file() {
  if [ -f "$1" ]; then
    return 0
  else
    return 1
  fi
}

check_file test.txt
echo $?   # prints exit status
```
## 5. Returning Output
- Use `echo` and capture output.
```bash
get_date() {
  echo "$(date +%Y-%m-%d)"
}

TODAY=$(get_date)
echo "$TODAY"
```

## 6. Local Variables
- Without `local`, variables become global.
```bash
my_func() {
  local NAME="DevOps"
  echo "$NAME"
}
```

## 7. Function Order Rule
- Functions must be defined before they are called.
```bash
greet() { echo "Hi"; }
greet
```

- **8. Example:**
```bash
check_service() {
  if systemctl is-active --quiet "$1"; then
    echo "$1 is running"
  else
    echo "$1 is not running"
  fi
}

check_service nginx
```

## 9. Using Main Function
```bash
main() {
  echo "Starting script"
}

main
```

## 10 Combining Functions

```bash
log_info() {
  echo "[INFO] $1"
}

log_error() {
  echo "[ERROR] $1"
}

log_info "Script started"
log_error "Something failed"
```
##  Reference Table

| Concept        | Syntax          |
| -------------- | --------------- |
| Define         | name() { }      |
| Call           | name            |
| Argument       | $1, $2          |
| Return code    | return 0        |
| Capture output | VAR=$(function) |
| Local variable | local VAR       |


---

# Task 5 – Text Processing Commands

## 1. grep – Search Patterns
- Search for text inside files.
**Useful Flags**
| Flag | Meaning                |
| ---- | ---------------------- |
| -i   | Ignore case            |
| -r   | Recursive search       |
| -c   | Count matches          |
| -n   | Show line numbers      |
| -v   | Invert match (exclude) |
| -E   | Extended regex         |

**Examples:**
- Case insensitive:
```bash
grep -i "error" file.log
```
- Count matches:
```bash
grep -c "ERROR" file.log
```
- Show line numbers:
```bash
grep -n "CRITICAL" file.log
```
- Exclude pattern:
```bash
grep -v "INFO" file.log
```
- Multiple patterns:
```bash
grep -E "ERROR|FAILED" file.log
```
- Recursive search:
```bash
grep -r "password" /var/www
```

## 2. awk – Column Processing

- **Print Columns**
```bash
awk '{print $1}' file
```

- **Field Separator**
Default separator = space
- Custom separator:
```bash
awk -F: '{print $1}' /etc/passwd
```
- Pattern Matching
```bash
awk '/ERROR/ {print $0}' file.log
```
- BEGIN / END Blocks
```bash
awk '
BEGIN {print "Start"}
{print $1}
END {print "End"}
' file
```
- **Example:**
Get disk usage column:
```bash
df -h | awk '{print $5}'
```

## 3. sed – Stream Editor
- Used for modifying text.
- **Substitute (Replace)**
```bash
sed 's/old/new/' file
```
Replace all:
```bash
sed 's/foo/bar/g' file
```
- **In-place Edit**
```bash
sed -i 's/foo/bar/g' file.txt
```
- **Delete Lines**
- Delete line 3:
```bash
sed '3d' file
```
- Delete matching pattern:
```bash
sed '/ERROR/d' file
```
- **Example:**
```bash
sed -i 's/port=80/port=8080/' app.conf
```

## 4. cut – Extract Columns

 - **By Delimiter**
```bash
cut -d: -f1 /etc/passwd
```
-d → delimiter
-f → field number

- **Extract Characters**
```bash
cut -c1-5 file.txt
```

## 5. sort – Sort Data

- Alphabetical:
```bash
sort file.txt
```
- Numerical:
```bash
sort -n file.txt
```
- Reverse:
```bash
sort -r file.txt
```
- Numeric reverse:
```bash
sort -rn file.txt
```
- Unique sorted:
```bash
sort file.txt | uniq
```
## 6. uniq – Remove Duplicates
- **⚠ Works only on sorted input.**

- Remove duplicates:
```bash
uniq file.txt
```
- Count occurrences:
```bash
uniq -c file.txt
```
- Sort by count:
```bash
sort file.txt | uniq -c | sort -rn
```

## 7. tr – Translate Characters

- Uppercase to lowercase:
```bash
echo "HELLO" | tr 'A-Z' 'a-z'
```
- Delete characters:
```bash
echo "hello123" | tr -d '0-9'
```
- Replace characters:
```bash
echo "a-b-c" | tr '-' ':'
```
## 8. wc – Word Count

- Lines:
```bash
wc -l file
```
- Words:
```bash
wc -w file
```
- Characters:
```bash
wc -c file
```
- Count lines in logs:
```bash
grep "ERROR" file.log | wc -l
```

## 9. head / tail

- First 10 lines:
```bash
head file.txt
```
- First 5 lines:
```bash
head -5 file.txt
```
- Last 10 lines:
```bash
tail file.txt
```
- Last 20 lines:
```bash
tail -20 file.txt
```
- Follow mode (real-time logs):
```bash
tail -f app.log
```

## Log Pattern
```bash
grep "ERROR" app.log \
| awk '{$1=$2=$3=""; print}' \
| sort \
| uniq -c \
| sort -rn \
| head -5
```

---

