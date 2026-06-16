# Git Commands Cheat Sheet

## Repository Commands

### Initialize a Git Repository

```bash
git init
```

### Check Repository Status

```bash
git status
```

### View Commit History

```bash
git log
```

### View Compact Commit History

```bash
git log --oneline
```

---

## Staging & Commit Commands

### Stage a Specific File

```bash
git add <file-name>
```

### Stage All Files

```bash
git add .
```

### Commit Changes

```bash
git commit -m "commit message"
```

### View Differences

```bash
git diff
```

---

## Branching Commands

### List Branches

```bash
git branch
```

### Create a New Branch

```bash
git branch feature-1
```

### Switch to a Branch

```bash
git switch feature-1
```

### Create and Switch to a Branch

```bash
git switch -c feature-2
```

### Switch Using Checkout

```bash
git checkout feature-1
```

### Delete a Branch

```bash
git branch -d feature-2
```

### Rename a Branch

```bash
git branch -m main
```

---

## Remote Repository Commands

### View Remote Repositories

```bash
git remote -v
```

### Add Remote Repository

```bash
git remote add origin <repository-url>
```

### Change Remote URL

```bash
git remote set-url origin <repository-url>
```

---

## Push Commands

### Push Main Branch

```bash
git push -u origin master
```

### Push Feature Branch

```bash
git push -u origin feature-1
```

### Push Existing Changes

```bash
git push
```

---

## Pull & Fetch Commands

### Download Changes Without Merging

```bash
git fetch origin
```

### Download and Merge Changes

```bash
git pull origin master
```

### Pull Latest Changes

```bash
git pull
```

---

## Clone & Fork Related Commands

### Clone a Repository

```bash
git clone <repository-url>
```

### Add Upstream Repository

```bash
git remote add upstream <repository-url>
```

### Fetch Upstream Changes

```bash
git fetch upstream
```

### Merge Upstream Changes

```bash
git merge upstream/master
```

---

## Useful Commands

### Show Current Branch

```bash
git branch
```

### Show All Branches and Commits

```bash
git log --oneline --all
```

### Show Tracked Remotes

```bash
git remote -v
```

---

## Key Concepts

| Term       | Description                                       |
| ---------- | ------------------------------------------------- |
| Repository | Storage location for project files and history    |
| Commit     | Snapshot of changes                               |
| Branch     | Independent line of development                   |
| HEAD       | Pointer to current branch/commit                  |
| Origin     | Default remote repository                         |
| Upstream   | Original repository from which a fork was created |
| Fetch      | Download changes without merging                  |
| Pull       | Fetch + Merge                                     |
| Clone      | Create local copy of repository                   |
| Fork       | Create personal copy of repository on GitHub      |
