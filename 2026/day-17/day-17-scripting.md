### 🧠 Day 17 – Shell Scripting - Day 2

**`for` Loop**
A loop means doing the same task again and again automatically.
Think like:
- Manually typing same command 10 times 
- Writing one loop to do it 10 times

**`for` loop**
- Takes a list (or numbers)
- Runs commands once for each item

## syntax

```bash
for item in list
do
  command
done
```
**Or short**
```bash
for item in list; do
  command
done
```

### my Task Loop through fruits

# Write a script

```bash
#!/bin/bash

for fruit in apple banana mango orange grapes
do
  echo "Fruit: $fruit"
done
```

# Output:
```text
Fruit: apple
Fruit: banana
Fruit: mango
Fruit: orange
Fruit: grapes
```

<img width="1193" height="177" alt="image" src="https://github.com/user-attachments/assets/4c0473c9-de19-42ab-9455-8c38dcf7ee71" />

**`fruit` is a variable Each loop assigns next value to `fruit` `echo` prints it**

## Print numbers 1 to 10

**Write a code**
```bash
#!/bin/bash

for num in {1..10}
do
  echo $num
done
```

# Output:

```text
1
2
3
...
10
```

<img width="1193" height="260" alt="image" src="https://github.com/user-attachments/assets/2b13ded3-732c-4dd2-b3c9-bf9d0ee85e5d" />

---

**`while` Loop**

# A while loop runs as long as a condition is true.
**Think like:**
- While battery > 0 → phone stays ON
- When battery = 0 → phone shuts down

**`while` loop syntax**

### my Task Countdown Script

# Write a script
```bash
#!/bin/bash

read -p "Enter a number: " NUM

while [ "$NUM" -ge 0 ]
do
  echo $NUM
  NUM=$((NUM - 1))
done

echo "Done!"
```
# Output:

```text
Enter a number: 5
5
4
3
2
1
0
Done!
```

<img width="1193" height="223" alt="image" src="https://github.com/user-attachments/assets/1c40d5bd-0367-4e65-bd23-c62841870443" />

## Condition check

```bash
[ "$NUM" -ge 0 ]
```

- `-ge` → greater than or equal

## Arithmetic operation

```bash
NUM=$((NUM - 1))
```

**This is how math works in shell.**

# my Common mistake (avoid this)

- Forgetting to update variable → infinite loop Always update the variable inside the loop.

---

### Command-Line Arguments

**Command-line arguments are values passed to a script when run it. Instead of hardcoding values inside the script, you pass them from outside.**

# Example:
```bash
./greet.sh Kishor
```
# Here:
`Kishor` is an argument

### Important argument variables

| Variable | Meaning                   |
| -------- | ------------------------- |
| `$0`     | Script name               |
| `$1`     | First argument            |
| `$2`     | Second argument           |
| `$#`     | Total number of arguments |
| `$@`     | All arguments             |


**my Task `greet.sh` (using $1)**

# Write a script
```bash
#!/bin/bash

if [ $# -eq 0 ]; then
  echo "Usage: ./greet.sh <name>"
  exit 1
fi

echo "Hello, $1!"
```

# Run without argument output:

```text
Usage: ./greet.sh <name>
```

# Run with argument:

```text
Hello, Kishor!
```

<img width="1193" height="146" alt="image" src="https://github.com/user-attachments/assets/cd90e3c7-fb5c-4dc5-832e-e5161806f63a" />

**my task `args_demo.sh`**

# Write a script
```bash
#!/bin/bash

echo "Script name: $0"
echo "Total arguments: $#"
echo "All arguments: $@"
```

# Output:
```text
Script name: ./args_demo.sh
Total arguments: 3
All arguments: one two three
```

<img width="1193" height="201" alt="image" src="https://github.com/user-attachments/assets/d1d43898-fbc8-4928-9fa4-c5e93072a7e1" />

---

### Install Packages via Script

# We’ll install these packages
```text
nginx
curl
wget
```

# Write a script
```bash
#!/bin/bash

PACKAGES="nginx curl wget"

for pkg in $PACKAGES
do
  if dpkg -s $pkg &> /dev/null; then
    echo "$pkg is already installed"
  else
    echo "Installing $pkg..."
    apt install -y $pkg
  fi
done
```

**Note: Run as root**
```bash
sudo ./install_packages.sh
```
# output

<img width="1193" height="201" alt="image" src="https://github.com/user-attachments/assets/f47592ca-d007-4352-9863-234f8674b077" />

# Loop 
```bash
for pkg in $PACKAGES
```
**Runs once for each package.**

# Check if installed
```bash
dpkg -s $pkg
```

# Final improved script
```bash
#!/bin/bash

if [ "$EUID" -ne 0 ]; then
  echo "Please run as root"
  exit 1
fi

PACKAGES="nginx curl wget"

for pkg in $PACKAGES
do
  if dpkg -s $pkg &> /dev/null; then
    echo "$pkg is already installed"
  else
    echo "Installing $pkg..."
    apt install -y $pkg
  fi
done
```

# output

<img width="1193" height="97" alt="image" src="https://github.com/user-attachments/assets/b943dcb5-6645-4fd8-93b5-3e8b6ecbebdf" />


---

### Error Handling in Shell Scripts

**`set -e` (Exit on error)** 
Means:
If any command fails → exit immediately

- **Task `safe_script.sh`**
# Write a script
```bash
#!/bin/bash
set -e

mkdir /tmp/devops-test || echo "Directory already exists"
cd /tmp/devops-test
touch test.txt

echo "All steps completed successfully"
```
# Output
```text
All steps completed successfully
```

<img width="1193" height="76" alt="image" src="https://github.com/user-attachments/assets/cd3838f3-35b6-4ae2-87e7-167893e1da57" />


# Run again:
```text
Directory already exists
All steps completed successfully
```

<img width="1193" height="166" alt="image" src="https://github.com/user-attachments/assets/c0ce7234-12dd-4ecd-b230-91a6e4a6fff9" />


- **`||` operator (OR)**
```bash
command || echo "Something went wrong"
```

# Meaning:
- If command fails → run the message

**Add error handling to install script `install_packages.sh`**

# Write a script
```bash
#!/bin/bash
set -e

# Check if script is run as root
if [ "$EUID" -ne 0 ]; then
  echo "Please run this script as root"
  exit 1
fi

# List of packages to install
PACKAGES="nginx curl wget"

for pkg in $PACKAGES
do
  if dpkg -s $pkg &> /dev/null; then
    echo "$pkg is already installed"
  else
    echo "Installing $pkg..."
    apt install -y $pkg
  fi
done

echo "All packages checked successfully"
```

# output

<img width="1193" height="166" alt="image" src="https://github.com/user-attachments/assets/38cba1d5-ffc9-48a0-927a-e9219a36b4bf" />


### set -e is used to stop script execution immediately if any command fails.

- Today I learned how to use loops and command-line arguments in shell scripting.
- I practiced writing scripts to install packages automatically and avoid repeated manual work.
- I understood how error handling (set -e) helps stop scripts safely when something goes wrong.
- These practices helped me write more reliable and reusable automation scripts.
