### Day 18 – Shell Scripting: Functions & Slightly Advanced Concepts

## call Functions Write a Scripts

**A function is just:**
- A small reusable block of code that does one specific job.

**Think like:**
- Instead of repeating same code 5 times
- Write it once as a function

🔹 Basic Function Syntax
```bash
function_name() {
  command
}
```

Or simply:
```bash
function_name() {
  echo "Hello"
}
```

## my Task 1: Basic Functions

**Create: `functions.sh`**

```bash
#!/bin/bash

greet() {
  echo "Hello, $1!"
}

add() {
  SUM=$(( $1 + $2 ))
  echo "Sum is: $SUM"
}

# Calling functions
greet "Kishor"
add 5 10
```

**Output:**
```text
Hello, Kishor!
Sum is: 15
```

<img width="1178" height="169" alt="image" src="https://github.com/user-attachments/assets/039f2778-409e-42b5-8766-e03d7956e7a0" />


- **Note: $1 and $2 inside function = arguments, called function like a command, Script became cleaner**

## my Task 2: Basic Functions

**Create: `disk_check.sh`**
```bash
#!/bin/bash

check_disk() {
  echo "Disk Usage:"
  df -h /
}

check_memory() {
  echo "Memory Usage:"
  free -h
}

main() {
  check_disk
  echo "---------------------"
  check_memory
}

main
```
**Output:**

<img width="1178" height="225" alt="image" src="https://github.com/user-attachments/assets/f3d1d028-d01c-4f0d-af5c-84975bcb4563" />


**Note:**
- Each function does one job
- `main()` controls flow

## my Task 3:  Strict Mode — set -euo pipefail
- Create `strict_demo.sh`
```bash
#!/bin/bash
set -euo pipefail

echo "Strict mode is enabled"

# 1️⃣ Undefined variable test (will fail because of -u)
# Uncomment the next line to test
# echo $UNDEFINED_VAR

# 2️⃣ Command failure test (will stop because of -e)
# Uncomment the next line to test
# false

# 3️⃣ Pipe failure test (will fail because of pipefail)
# Uncomment the next line to test
# cat missing_file.txt | grep "DevOps"

echo "Script completed successfully"
```
**Output:**

```text
Strict mode is enabled
Script completed successfully
```

<img width="1178" height="130" alt="image" src="https://github.com/user-attachments/assets/c5c9ce8f-5939-4cd5-835b-8eedcb03bde4" />

## my tast 4 
- creat a script `exit_demo.sh` (Exit on Error)
```bash
#!/bin/bash
set -e

echo "Script started"

echo "Creating directory..."
mkdir /tmp/devops-test

echo "Trying to create same directory again..."
mkdir /tmp/devops-test   # This will fail

echo "This line will NOT execute"
```

**Output:**
```text
This line will NOT execute
```

<img width="1178" height="166" alt="image" src="https://github.com/user-attachments/assets/18ebc89b-b1dc-4676-93ba-9d9b01d11d49" />


**Note: `-e` makes script stop when any command fails.**

## my tast 4 
- creat a script `-u` (Undefined Variable Error)
```bash
#!/bin/bash
set -u

echo "Script started"

NAME="Kishor"
echo "Hello $NAME"

echo "Trying to print undefined variable..."
echo $AGE   # AGE is not defined

echo "This line will NOT execute"
```

**Output:**
```text
Script started
Hello Kishor
Trying to print undefined variable...
AGE: unbound variable
```

<img width="1178" height="166" alt="image" src="https://github.com/user-attachments/assets/6e101983-c391-4065-be97-d4fe392a291d" />


**Note: `-u` Stops if variable is not defined**

### What it means:
| Flag          | Meaning                          |
| ------------- | -------------------------------- |
| `-e`          | Exit if any command fails        |
| `-u`          | Error if undefined variable used |
| `-o pipefail` | Fail if any part of pipe fails   |

## Professionals usually combine both:
```bash
set -eu
```
and
```bash
set -euo pipefail
```

## my tast 4 Advanced Script
- creat a script `system_info.sh`
```bash
#!/bin/bash
set -euo pipefail

print_header() {
  echo "=============================="
  echo "$1"
  echo "=============================="
}

system_info() {
  print_header "System Info"
  hostname
  uname -a
}

uptime_info() {
  print_header "Uptime"
  uptime
}

disk_info() {
  print_header "Disk Usage"
  df -h | head -5
}

memory_info() {
  print_header "Memory Usage"
  free -h
}

main() {
  system_info
  uptime_info
  disk_info
  memory_info
}

main
```

**Output:**
```text
System Info
==============================
ip-172-31-27-38
Linux ip-172-31-27-38 6.14.0-1018-aws #18~24.04.1-Ubuntu SMP Mon Nov 24 19:46:27 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
==============================
Uptime
==============================
 06:42:20 up 49 min,  1 user,  load average: 0.00, 0.00, 0.00
==============================
Disk Usage
==============================
Filesystem       Size  Used Avail Use% Mounted on
/dev/root        6.8G  2.3G  4.5G  33% /
tmpfs            458M     0  458M   0% /dev/shm
tmpfs            183M  888K  182M   1% /run
tmpfs            5.0M     0  5.0M   0% /run/lock
==============================
Memory Usage
==============================
               total        used        free      shared  buff/cache   available
Mem:           914Mi       351Mi       256Mi       2.7Mi       469Mi       562Mi
Swap:             0B          0B          0B
```

<img width="1178" height="474" alt="image" src="https://github.com/user-attachments/assets/ded10f9d-6fe0-453e-b4a9-bc9dd044df76" />


### What i Learned Today

- Functions make scripts reusable
- `main()` keeps script clean
- `set -euo pipefail` makes script safe
- `local` avoids variable conflicts
- Real-world script structuring
