# Git Commands Reference

This file contains Git commands that I learned and practiced during my DevOps journey.

---

## Setup & Configuration

### Check Git Version


git --version


Used to check whether Git is installed and verify the version.

### Configure Username


git config --global user.name "your-name"


Sets the username for Git commits.

### Configure Email


git config --global user.email "your-email@example.com"


Sets the email address for Git commits.

### View Configuration


git config --list

Displays all configured Git settings.

---

## Repository Commands

### Initialize Repository


git init


Creates a new Git repository in the current directory.

### Check Repository Status


git status


Shows the current state of the repository, including modified and untracked files.

---

## Staging Changes

### Add Specific File


git add filename


Adds a specific file to the staging area.

Example:


git add git-commands.md


### Add All Changes


git add .


Adds all modified and new files to the staging area.

---

## Commit Changes

### Create Commit


git commit -m "commit message"


Saves staged changes to the repository.

Example:


git commit -m "Initial commit"


---

## Viewing History

### View Commit History


git log


Displays detailed commit history.

### Compact Commit History


git log --oneline


Displays commit history in a short format.

---

## Useful Linux Commands Used

### Create Directory


mkdir devops-git-practice


Creates a new project directory.

### Change Directory


cd devops-git-practice


Moves into the project directory.

### List Files


ls -la


Displays all files, including hidden files such as `.git`.

### Create File


touch git-commands.md


Creates a new file.

---

## Notes

* Git tracks changes in files over time.
* The staging area acts as a middle step before committing.
* The .git folder stores all repository information.
* Deleting the .git folder removes Git tracking and commit history.

---


