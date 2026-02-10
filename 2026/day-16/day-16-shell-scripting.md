### 🧠 Day 16 – Shell Scripting

## my First Script

**A shell script is just a file that contains Linux commands written in order. Instead of typing commands again and again, we automate them.**

Think like:
- Manual work → Script → Automation

**Shebang?** (`#!/bin/bash`)
The shebang tells Linux:
- “Which program should run this file?”
``` bash
#!/bin/bash
```

Means:
- Use bash shell to run this script
Without shebang:
- Linux doesn’t know how to interpret the file

### Your FIRST script (hands-on)

**Create a file**
```bash
touch hello.sh
```

**Open the file**
```bash
vim hello.sh
```
- **Note:** In Vim, press `I` to enter insert mode and start editing.
- **Note:** You can also edit the file using `nano hello.sh`. To save and exit, press `Ctrl + O`, then Enter, and then `Ctrl + X`.

**Write a code**
```bash
#!/bin/bash
echo "Hello, DevOps!"
```
<img width="1130" height="96" alt="image" src="https://github.com/user-attachments/assets/4c5d41bc-32c7-4c17-b554-7218e37254e7" />

- **Note:** After completing the script editing, press `Esc`, then type `:wq` and press Enter to save and exit Vim.

## Make the script executable

```bash
chmod +x hello.sh
     or
chmod 777 hello.sh
```

## Run the script

```bash
./hello.sh
```

## Output:
```text
Hello, DevOps!
```

<img width="1130" height="96" alt="image" src="https://github.com/user-attachments/assets/46631c14-9e5b-4ba9-8757-558f48512561" />

---

**Variables & `echo` (Core Basics)**

- A variable is a place where you store a value so you can reuse it later.

**Think like:**
- Variable = container with a label
**Example:**

```bash
  NAME="Kishor"
```
**Now `NAME` holds the value `Kishor`.**

## Rules for variables in shell

No space around `=`
`NAME = Kishor` → wrong
`NAME=Kishor` → correct

**Variable names are usually UPPERCASE (best practice)**

**I Create your second script:** `variables.sh`

## Write a code

```bash
#!/bin/bash

NAME="Kishor"
ROLE="DevOps Engineer"

echo "Hello, I am $NAME and I am a $ROLE"
```

**Output:**
```test
Hello, I am Kishor and I am a DevOps Engineer
```

<img width="1130" height="96" alt="image" src="https://github.com/user-attachments/assets/f2762d46-9ba3-4ea9-a500-cd1e978d25c5" />


### $ sign (very important)

`$` is used to access the value of a variable.

```bash
$NAME
$ROLE
```
Without `$`, shell thinks it’s just text.

### Single quotes vs Double quotes

## Double quotes
```bash
echo "Hello $NAME"
```

**Output:**
```text
Hello Kishor
```

## Single quotes
```bash
echo 'Hello $NAME'
```

**Output:**
```text
Hello $NAME
```

**Note:**
- Double quotes → variables are expanded Single quotes → treated as plain text
- Use double quotes when you want variable values.

## Real example

```bash
SERVICE="nginx"
echo "Checking status of $SERVICE"
```
**This makes scripts dynamic and reusable.**

---

**User Input with `read`**
- I Create a new script: `greet.sh`

## Write a code

```bash
#!/bin/bash

read -p "Enter your name: " NAME
read -p "Enter your favourite tool: " TOOL

echo "Hello $NAME, your favourite tool is $TOOL"
```
**Example run:**
```text
Enter your name: Kishor
Enter your favourite tool: Docker
Hello Kishor, your favourite tool is Docker
```

<img width="1130" height="130" alt="image" src="https://github.com/user-attachments/assets/d9a6d189-17ad-43d1-bc41-bb55396a7157" />

- **`-p` flag**
```bash
read -p "Enter name: " NAME
```
- `p` → shows prompt message
- `NAME` → variable to store input

**Common mistake**
- Forgetting variable name:
```bash
  read -p "Enter name: "
```
- Using $ while reading:
```bash
read -p "Enter name: " $NAME
```
- Correct:
```bash
read -p "Enter name: " NAME
```

---

**`if–else` Conditions (Decision Making)**

**`if–else` means:**
- `If` a condition is true → do this
- `Else` → do something else

**Just like real life:**
- If it’s raining → take umbrella
- Else → go normally

**`if` syntax**

```bash
if [ condition ]; then
  command
else
  command
fi
```

**Note: VERY IMPORTANT** → `[ condition ]` → space before and after `[ and ]`

**positive / negative / zero**

- I Create a new script: `number.sh`
## Write a code

```bash
#!/bin/bash

read -p "Enter a number: " NUM

if [ "$NUM" -gt 0 ]; then
  echo "The number is positive"
elif [ "$NUM" -lt 0 ]; then
  echo "The number is negative"
else
  echo "The number is zero"
fi
```
**Output:**

```text
I Enter a number: -5
The number is negative
&
Enter a number: 5
The number is positive
```

<img width="1130" height="168" alt="image" src="https://github.com/user-attachments/assets/3cf43adc-0fdb-40ab-8e8d-aa0d740eb450" />

**Important operators**

| Operator | Meaning      |
| -------- | ------------ |
| -gt      | greater than |
| -lt      | less than    |
| -eq      | equal        |
| -ne      | not equal    |

## File exists or not

- I Create a new script: `check.sh`
## Write a code

```bash
#!/bin/bash

read -p "Enter filename: " FILE

if [ -f "$FILE" ]; then
  echo "File exists"
else
  echo "File does not exist"
fi
```

**Output:**

<img width="1130" height="127" alt="image" src="https://github.com/user-attachments/assets/e8329599-1990-4837-b6d8-908fbd366603" />

**`-f` meaning**
| Flag | Meaning                  |
| ---- | ------------------------ |
| -f   | File exists              |
| -d   | Directory exists         |
| -e   | File or directory exists |

---

### Combine It All ###

- variables
- read
- if–else
- real system command

## Goal of the script

**We will create server_check.sh that:**
- Stores a service name in a variable
- Asks user if they want to check status
- If y → checks service status
- If n → skips

- I Create a new script: `server_check.sh`
## Write a code

```bash
#!/bin/bash

SERVICE="ssh"

read -p "Do you want to check the status of $SERVICE? (y/n): " CHOICE

if [ "$CHOICE" = "y" ]; then
  systemctl is-active --quiet $SERVICE

  if [ $? -eq 0 ]; then
    echo "$SERVICE service is running"
  else
    echo "$SERVICE service is NOT running"
  fi

elif [ "$CHOICE" = "n" ]; then
  echo "Skipped."
else
  echo "Invalid input. Please enter y or n."
fi
```

**Example run:**
```text
Do you want to check the status of ssh? (y/n): y
ssh service is running
```

<img width="1130" height="127" alt="image" src="https://github.com/user-attachments/assets/941a6bdc-ea18-4fa5-862b-6680775b356a" />

## Important things

**Variable**
```bash
SERVICE="ssh"
```
- I change this to:
```bash
SERVICE="nginx"
```
**Output:**

<img width="1130" height="189" alt="image" src="https://github.com/user-attachments/assets/53bbb40c-9f05-4d19-ab7a-7b014333be2f" />

### $? (IMPORTANT)

```bash
$?
```

**`$?`**
- Stores exit status of last command
- 0 → success
- non-zero → failure

---

### Today I learning **Shell Scripting fundamentals**, which are essential for automation and daily DevOps tasks. I focused on understanding the basics and writing simple scripts.###

---
