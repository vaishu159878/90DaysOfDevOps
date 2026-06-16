# Day 24 - Advanced Git Operations

## Task 1: Git Merge

### What I Did

* Created `feature-login` branch and added commits.
* Merged it into `main`.
* Git performed a Fast-Forward Merge because `main` had not moved ahead.
* Created `feature-signup` branch and added commits.
* Added a commit on `main` before merging.
* Merged `feature-signup` into `main`.
* Git created a Merge Commit because both branches had different histories.

### Answers

#### What is a Fast-Forward Merge?

A fast-forward merge happens when there are no new commits on the target branch. Git simply moves the branch pointer forward.

#### When does Git create a Merge Commit?

Git creates a merge commit when both branches have unique commits and their histories have diverged.

#### What is a Merge Conflict?

A merge conflict occurs when the same part of a file is modified in different branches and Git cannot decide which change to keep automatically.

---

## Task 2: Git Rebase

### What I Did

* Created `feature-dashboard`.
* Added multiple commits.
* Added a new commit on `main`.
* Rebases `feature-dashboard` onto `main`.
* Observed the commit graph using:

```bash
git log --oneline --graph --all
```

### Answers

#### What does rebase actually do?

Rebase moves commits from one branch and reapplies them on top of another branch.

#### How is the history different from a merge?

Rebase creates a cleaner and linear history while merge preserves branch history.

#### Why should you never rebase shared commits?

Because rebase rewrites commit history. Other developers may face conflicts and synchronization issues.

#### When would you use rebase vs merge?

* Rebase: To keep history clean before merging.
* Merge: To preserve complete branch history.

---

## Task 3: Squash Merge vs Regular Merge

### What I Did

* Created `feature-profile` branch.
* Added multiple small commits.
* Merged using `--squash`.
* Created `feature-settings`.
* Merged normally.

### Answers

#### What does squash merging do?

It combines multiple commits into a single commit before merging.

#### When would you use squash merge?

When a feature branch contains many small commits that are not useful individually.

#### When would you use a regular merge?

When preserving complete development history is important.

#### What is the trade-off of squashing?

The detailed commit history is lost.

---

## Task 4: Git Stash

### What I Did

* Made changes without committing.
* Used stash to save work.
* Switched branches.
* Restored changes.
* Created multiple stashes.
* Applied a specific stash.

### Commands Used

```bash
git stash push -m "First stash"
git stash list
git stash apply stash@{0}
```

### Answers

#### Difference between git stash pop and git stash apply

`git stash pop`

* Applies the stash.
* Removes it from stash list.

`git stash apply`

* Applies the stash.
* Keeps it in stash list.

#### When would you use stash in real projects?

When switching tasks quickly without committing incomplete work.

---

## Task 5: Cherry Pick

### What I Did

* Created a branch with multiple commits:

  * fix 1
  * fix 2
  * fix 3
* Switched to `main`.
* Cherry-picked only the second commit.
* Faced a conflict and resolved it manually.
* Continued cherry-pick successfully.

### Commands Used

```bash
git cherry-pick <commit-id>
git status
git add new.txt
git cherry-pick --continue
```

### Answers

#### What does cherry-pick do?

Cherry-pick copies a specific commit from one branch and applies it to another branch.

#### When would you use cherry-pick?

When only a specific bug fix or feature commit is needed without merging the entire branch.

#### What can go wrong with cherry-picking?

* Merge conflicts
* Duplicate commits
* Confusing project history

---

## Mistakes I Made

* Tried applying a stash that did not exist.
* Used `git show head` instead of `git show HEAD`.
* Faced a cherry-pick conflict and resolved it.
* Initially got confused while reading the commit graph.


