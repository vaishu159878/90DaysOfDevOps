# Day 22 - Introduction to Git

## Overview

Today I started learning Git, which is one of the most important tools in DevOps and software development.

The goal of this task was to understand the basics of version control, create my first Git repository, and learn how Git tracks changes in files.

---

## What I Learned

* What Git is and why it is used
* How to configure Git with username and email
* How to initialize a Git repository
* Understanding the Git workflow

  * Working Directory
  * Staging Area
  * Repository
* How to stage and commit changes
* How to view commit history
* Importance of the .git directory

---

## Commands Practiced


git --version
git config --global user.name "username"
git config --global user.email "email@example.com"

git init
git status

git add git-commands.md
git commit -m "Initial commit"

git log
git log --oneline

---


## 1. What is the difference between git add and git commit?

git add is used to move changes from the working directory to the staging area.

git commit is used to save the staged changes permanently in the Git repository.

In simple words, git add prepares the changes and git commit saves them.

---

## 2. What does the staging area do? Why doesn't Git just commit directly?

The staging area acts as a middle step between editing files and committing them.

It allows us to choose which changes we want to include in a commit.

Git does not commit directly because sometimes we may modify many files but only want to save specific changes. The staging area gives us more control before creating a commit.

---

## 3. What information does git log show you?

The git log command shows the commit history of a repository.

It displays:

* Commit ID (hash)
* Author name
* Date and time of commit
* Commit message

It helps track what changes were made and when they were made.

---

## 4. What is the .git/ folder and what happens if you delete it?

The .git folder is the hidden directory that stores all Git information for a repository.

It contains:

* Commit history
* Branch information
* Configuration settings
* Repository metadata

If the .git folder is deleted, the project will no longer be a Git repository and all commit history will be lost.

---

## 5. What is the difference between a working directory, staging area, and repository?

### Working Directory

The place where I create, edit, and delete files.

### Staging Area

A temporary area where changes are prepared before committing.

### Repository

The location where committed changes and project history are stored permanently.

Workflow:

Working Directory → Staging Area → Repository

---

## Screenshots

img.png
img1.png
img2.png
img3.png
img4.png

