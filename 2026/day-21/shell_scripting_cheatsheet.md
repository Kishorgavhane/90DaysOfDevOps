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

