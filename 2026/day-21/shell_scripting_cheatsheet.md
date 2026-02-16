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

