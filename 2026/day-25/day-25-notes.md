# Day 25 – Git Reset vs Revert & Branching Strategies

## Task 1: Git Reset – Hands-On

### Commit History Created

```bash
Commit A
Commit B
Commit C
```

### 1. git reset --soft HEAD~1

```bash
git reset --soft HEAD~1
```

#### Observation

* Last commit is removed from Git history.
* Changes remain in the Staging Area.
* Files are ready to commit again.

#### Status

```bash
git status
```

Shows:

```bash
Changes to be committed
```

---

### 2. git reset --mixed HEAD~1

```bash
git reset --mixed HEAD~1
```

#### Observation

* Last commit is removed.
* Changes remain in working directory.
* Files are unstaged.

#### Status

```bash
git status
```

Shows:

```bash
Changes not staged for commit
```

---

### 3. git reset --hard HEAD~1

```bash
git reset --hard HEAD~1
```

#### Observation

* Last commit removed.
* Staging area cleared.
* Working directory reverted.
* Changes permanently deleted (unless recovered using reflog).

#### Status

```bash
Working tree clean
```

---

## Reset Types Comparison

| Command | Commit Removed | Changes Kept | Staged |
| ------- | -------------- | ------------ | ------ |
| --soft  | Yes            | Yes          | Yes    |
| --mixed | Yes            | Yes          | No     |
| --hard  | Yes            | No           | No     |

### Which one is destructive?

`git reset --hard`

Reason:

* Deletes commits and working directory changes.
* Data may be lost permanently if not recoverable through reflog.

### When to use each?

#### Soft Reset

Use when:

* Commit message is wrong.
* Want to combine commits.

#### Mixed Reset

Use when:

* Need to modify files before recommitting.

#### Hard Reset

Use when:

* Want to completely discard changes.
* Return repository to a known state.

### Should you use reset on pushed commits?

Generally No.

Reason:

* Rewrites commit history.
* Can break other developers' work.
* Requires force push.

---

# Task 2: Git Revert – Hands-On

### Commit History

```bash
Commit X
Commit Y
Commit Z
```

### Revert Commit Y

```bash
git revert <commit-id-of-Y>
```

### Observation

Git creates a new commit:

```bash
Revert "Commit Y"
```

The changes introduced by Y are undone.

---

### Is Commit Y still in history?

Yes.

```bash
git log
```

Example:

```bash
Commit Revert Y
Commit Z
Commit Y
Commit X
```

Commit Y remains in history.

---

## Revert Questions

### How is git revert different from git reset?

Git Reset:

* Moves branch pointer backward.
* Can remove commits from history.

Git Revert:

* Creates a new commit that undoes previous changes.
* Preserves history.

### Why is revert safer?

Because:

* History remains intact.
* No force push required.
* Team members are not affected.

### When to use revert vs reset?

Use Reset:

* Local commits.
* Cleanup before pushing.

Use Revert:

* Shared branches.
* Production fixes.
* Already pushed commits.

---

# Task 3: Reset vs Revert Summary

| Feature                  | git reset           | git revert          |
| ------------------------ | ------------------- | ------------------- |
| What it does             | Moves HEAD backward | Creates undo commit |
| Removes history          | Yes                 | No                  |
| Creates new commit       | No                  | Yes                 |
| Safe for pushed branches | No                  | Yes                 |
| Requires force push      | Often               | No                  |
| Best use case            | Local cleanup       | Shared repositories |

---

# Task 4: Branching Strategies

## 1. GitFlow

### How it Works

Uses multiple long-lived branches:

```text
main
│
├── develop
│    ├── feature/login
│    ├── feature/signup
│
├── release
│
└── hotfix
```

### Used For

* Enterprise applications
* Scheduled releases

### Pros

* Structured workflow
* Supports release cycles
* Easy hotfix management

### Cons

* Complex
* Many branches to manage

---

## 2. GitHub Flow

### How it Works

```text
main
│
├── feature-login
├── feature-payment
└── feature-ui
```

Create PR → Review → Merge into main.

### Used For

* SaaS products
* Continuous deployment

### Pros

* Simple
* Fast delivery
* Easy collaboration

### Cons

* Less control over releases

---

## 3. Trunk-Based Development

### How it Works

```text
main (trunk)
│
├── short-lived branch
└── merge quickly
```

Developers merge small changes frequently.

### Used For

* Google
* Facebook
* Modern DevOps teams

### Pros

* Faster integration
* Fewer merge conflicts
* Supports CI/CD

### Cons

* Requires strong testing
* Discipline needed

---

## Answers

### Which strategy for a startup shipping fast?

GitHub Flow

Reason:

* Simple workflow.
* Fast deployments.
* Minimal process overhead.

### Which strategy for a large team with scheduled releases?

GitFlow

Reason:

* Better release management.
* Dedicated release and hotfix branches.

### Which one does Kubernetes use?

Kubernetes primarily follows a GitHub Flow style with release branches for version management.

---

# Task 5: Important Commands Added

## Reset

```bash
git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1
```

## Revert

```bash
git revert <commit-id>
```

## Recovery

```bash
git reflog
```

## Branching

```bash
git branch
git switch
git checkout
```

## Merge

```bash
git merge branch-name
```

## Rebase

```bash
git rebase main
```

## Stash

```bash
git stash
git stash pop
git stash list
```

## Cherry Pick

```bash
git cherry-pick <commit-id>
```

---

