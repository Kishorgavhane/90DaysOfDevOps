# Day 22: Git

---

# What is Git?

**Git is a Distributed Version Control System (DVCS).**

Let’s break this:
- **Version Control** → Tracks changes in files over time
- **Distributed** → Every developer has a complete copy of the repository

## Git importance 

- Code is stored in Git
- CI/CD pipelines trigger from Git commits
- Infrastructure as Code (Terraform, Ansible) stored in Git
- Kubernetes YAML files stored in Git
- Collaboration happens through Git

# Git Architecture

`Working Directory → Staging Area → Repository`

## 1. Working Directory

- This is my actual project folder.
- Where i create and edit files.
- Git watches changes here.

**Example:**
```text
git-commands.md
day-22-notes.md
```

## 2.Staging Area (Index)

- Temporary holding area.
- I select what should go into the next commit.
- Controlled by: `git add`

Think of it like:
- Shopping cart before billing.

## 3.Repository (.git folder)

- Where commits are permanently stored.
- Controlled by: `git commit`
- Inside hidden `.git/` directory.

If delete `.git/` → your Git history is gone.

# Install & Configure Git

- **Step 1: Verify Git**
```bash
git --version
```

<img width="1126" height="76" alt="image" src="https://github.com/user-attachments/assets/459bfc1a-daf8-4530-9121-75fe43339675" />


- **Step 2: Configure Identity**
```bash
git config --global user.name "Kishor"
git config --global user.email "kishor@email.com"
```

**Every commit stores:**

- Author name
- Author email
- Timestamp
- Commit message
- Unique hash

Check config:
```bash
git config --list
```

<img width="1126" height="87" alt="image" src="https://github.com/user-attachments/assets/43023f36-c439-4786-a0bd-7268f531b283" />


# Create First Repository

- **Step 1: Create Folder**
```bash
mkdir devops-git-practice
cd devops-git-practice
```

- **Step 2: Initialize Git**
```bash
git init
```

### What happens internally?

- Creates hidden `.git/` folder
- Sets default branch (main/master)
- Initializes empty repository

<img width="1126" height="286" alt="image" src="https://github.com/user-attachments/assets/a0b9ba03-272a-4523-b002-21d7c58b8127" />


- **Inside `.git:`**
- HEAD
- config
- objects/
- refs/
This is the heart of Git.
If deleted → no version control anymore.

# Git Workflow

- **Create File**
```bash
touch git-commands.md
```
- **Check status:**
```bash
git status
```

<img width="1126" height="233" alt="image" src="https://github.com/user-attachments/assets/74b6ae85-0d04-44a0-a280-480ab8663035" />


- **Output will show:**
Untracked files
- **Meaning:**
Git sees file but is not tracking yet.

**Stage File**
```bash
git add git-commands.md
```

Now file moves to:
Working Directory → Staging Area

Check:
```bash
git status
```

<img width="1126" height="223" alt="image" src="https://github.com/user-attachments/assets/20974c19-c099-4288-8451-f9c92c62e646" />


It will show:
- Changes to be committed

**Commit**
```bash
git commit -m "Added initial git commands file"
```

**Now:**
- Staging Area → Repository

<img width="1126" height="161" alt="image" src="https://github.com/user-attachments/assets/5d04eaca-cf22-482e-ac04-08ea444f929d" />


# git log

```bash
git log
```

**Shows:**
- Commit hash
- Author
- Date
- Message

<img width="1126" height="161" alt="image" src="https://github.com/user-attachments/assets/d346bab3-3551-4821-8734-9895d49b6b58" />


**Compact format:**
```bash
git log --oneline
```

<img width="1126" height="69" alt="image" src="https://github.com/user-attachments/assets/77f9385d-af16-4ec7-8231-4da8928a5784" />


**Example:**
```bash
85ea1f5 Added initial git commands file
b72ac98 Updated git commands
```
**Each commit has:**
- 🔑 Unique SHA-1 hash
- 🧑 Author
- 📅 Timestamp
- 📝 Message

# Difference Between Important Commands

- **git add vs git commit**


| git add               | git commit                 |
| --------------------- | -------------------------- |
| Moves file to staging | Saves snapshot permanently |
| Prepares changes      | Records changes            |
| Temporary             | Permanent                  |


**What does staging area do? Why not commit directly?**

Because:
- may be change 10 files
- But want to commit only 2
- Staging area gives control.

**What is git log?**

Shows:
- Full history
- Who changed what
- When it changed
- Commit messages

**What is .git folder?**
- Stores commits
- Stores branches
- Stores config
- Stores objects


### Working Directory vs Staging vs Repository

| Area              | Purpose                   |
| ----------------- | ------------------------- |
| Working Directory | Where you edit files      |
| Staging Area      | Where you prepare commits |
| Repository        | Where history is stored   |


---
🔹 Understood Working Directory, Staging Area, Repository
🔹 Created my first repository
🔹 Explored the hidden .git folder
🔹 Built my own Git commands reference
---
