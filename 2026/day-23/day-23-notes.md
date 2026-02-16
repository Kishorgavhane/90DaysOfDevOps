# Day 23 – Git Branching & Working with GitHub

### Branch Definition 

- **A branch in Git is simply a pointer to a commit.** That’s it.

**It is NOT:**
- A copy of files
- A duplicate folder
- A separate repository

It is just a movable label pointing to a commit.

## Visual Understanding

**made commits:**
```text
A --- B --- C   (main)
```
**Now I create a branch:**
```text
A --- B --- C   (main)
              \
               D --- E   (feature-1)
```

- **`feature-1` starts from the current commit.**

Branches `allow`:
- Feature development
- Bug fixes
- Experiments
- Team collaboration
Without affecting `master`.

## Why Not Commit Everything to main?

**If I commit everything to main:**

- Code becomes unstable
- Bugs affect production
- Hard to test features separately
- No clean history

**Best Practice:**

- main = stable
- feature branches = development
- merge after testing

## HEAD?

HEAD is a pointer to the current branch.

Example:

If me are on main:
```bash
HEAD → master → commit C
```

If i switch:
```bash
HEAD → feature-1 → commit E
```

**HEAD always tells Git: 👉 “Where am I currently working?”**

## What Happens When i Switch Branches?

**When i run:**
```bash
git switch feature-1
```

**Git:**

- Moves HEAD to feature-1
- Updates your working directory to match that branch
- Restores files exactly as they were in that commit

**This is NOT copying files. Git is reconstructing snapshot from commit.**

## Hands-On Branching Commands

## 1. List All Branches
```bash
git branch
```
Output:
```bash
* master
```


<img width="1161" height="69" alt="image" src="https://github.com/user-attachments/assets/5fdfac4c-aefa-4ac2-bf73-42d2dc3a37c0" />


- `*` means current branch.

## 2. Create New Branch

```bash
git branch feature-1
```

This:

- Creates pointer
- Does NOT switch

## 3. Switch to Branch

```bash
git switch feature-1
```


<img width="1161" height="173" alt="image" src="https://github.com/user-attachments/assets/3b86b48a-6855-43b7-aa98-4c6a08c4fd86" />



**Now HEAD → feature-1**

## 4. Create and Switch in One Command

```bash
git switch -c feature-2
```


<img width="1161" height="127" alt="image" src="https://github.com/user-attachments/assets/703fac80-19f1-4639-b508-43930add4f89" />


## 5. Make a Commit in feature-1

```bash
echo "Feature 1 update" >> git-commands.md
git add .
git commit -m "Added feature-1 changes"
```

**Now: `feature-1` has new commit. main does NOT.**

## 6. Switch Back to main
```bash
git switch main
```

<img width="1161" height="177" alt="image" src="https://github.com/user-attachments/assets/015e03eb-d3c9-4628-bcd8-b71d2b1a9cbd" />



- **Now check file: The feature changes will NOT be there.**
- **Why?_Because main does not contain that commit.**

## 7. Delete Branch

Safe delete:
```bash
git branch -d feature-2
```

Force delete:
```bash
git branch -D feature-2
```

<img width="1161" height="128" alt="image" src="https://github.com/user-attachments/assets/47e735e3-dbce-4ab1-8b94-a422ab1763c6" />


