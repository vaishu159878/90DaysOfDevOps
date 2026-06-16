# Day 23 - Git Branching & Working with GitHub

## Task 1: Understanding Branches

### 1. What is a branch in Git?

A branch in Git is an independent line of development. It allows developers to work on new features, bug fixes, or experiments without affecting the main codebase.

### 2. Why do we use branches instead of committing everything to main?

Branches help keep the main branch stable. Developers can test changes safely and merge them into main only after they are verified.

### 3. What is HEAD in Git?

HEAD is a pointer that refers to the current branch and latest commit you are working on.

### 4. What happens to your files when you switch branches?

Git updates the working directory to match the selected branch. Files may appear, disappear, or change depending on the commits present in that branch.

---

## Task 2: Branching Commands

### List all branches

```bash
git branch
```

### Create a new branch

```bash
git branch feature-1
```

### Switch to feature-1

```bash
git switch feature-1
```

### Create and switch to a new branch in one command

```bash
git switch -c feature-2
```

### Difference between git switch and git checkout

* `git switch` is used specifically for switching branches.
* `git checkout` can switch branches and restore files.
* `git switch` is easier and safer for branch operations.

### Verify branch isolation

I created a commit on `feature-1`. After switching back to `main`, the commit was not visible because each branch maintains its own commit history until merged.

### Delete a branch

```bash
git branch -d feature-2
```

---

## Task 3: Push to GitHub

### Add remote repository

```bash
git remote add origin https://github.com/<username>/<repository-name>.git
```

### Verify remote

```bash
git remote -v
```

### Push main branch

```bash
git push -u origin main
```

### Push feature-1 branch

```bash
git push -u origin feature-1
```

### Difference between origin and upstream

**origin**

* The default remote repository.
* Usually points to your own GitHub repository.

**upstream**

* Refers to the original repository from which a fork was created.
* Used to keep your fork updated with changes from the original project.

---

## Task 4: Pull from GitHub

### Pull latest changes

```bash
git pull origin main
```

### Difference between git fetch and git pull

**git fetch**

* Downloads new commits from the remote repository.
* Does not modify the local branch.

**git pull**

* Downloads changes and automatically merges them into the current branch.
* Equivalent to `git fetch` + `git merge`.

---

## Task 5: Clone vs Fork

### What is the difference between clone and fork?

**Clone**

* Creates a local copy of a repository on your computer.
* Git operation.

**Fork**

* Creates a copy of someone else's repository under your GitHub account.
* GitHub feature.

### When would you clone vs fork?

**Clone**

* When you have direct access to the repository.
* For personal projects or team projects.

**Fork**

* When contributing to open-source projects where you don't have write access.

### How do you keep your fork in sync with the original repository?

Add the original repository as an upstream remote:

```bash
git remote add upstream <original-repository-url>
```

Fetch updates:

```bash
git fetch upstream
```

Merge updates into main:

```bash
git checkout main
git merge upstream/main
```
