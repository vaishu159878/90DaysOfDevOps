# Git Commands

## Git Merge

### Fast Forward Merge

```bash
git merge feature-login
```

Merges a branch when there are no new commits on the target branch.

### Merge Commit

```bash
git merge feature-signup
```

Creates a merge commit when both branches have different commit histories.

---

## Git Rebase

### Rebase Current Branch onto Main

```bash
git rebase main
```

Moves current branch commits on top of the latest main branch.

### Continue Rebase

```bash
git rebase --continue
```

Continue rebase after resolving conflicts.

### Abort Rebase

```bash
git rebase --abort
```

Cancel the rebase operation.

---

## Squash Merge

### Squash Multiple Commits

```bash
git merge --squash feature-profile
```

Combines all commits into a single commit before merging.

---

## Git Stash

### Create Stash

```bash
git stash
```

Save tracked changes temporarily.

### Stash with Message

```bash
git stash push -m "First stash"
```

Save changes with a custom message.

### Include Untracked Files

```bash
git stash -u
```

Save tracked and untracked files.

### List Stashes

```bash
git stash list
```

Display all saved stashes.

### Apply Stash

```bash
git stash apply stash@{0}
```

Apply a specific stash without deleting it.

### Pop Stash

```bash
git stash pop
```

Apply and remove the latest stash.

### Remove Stash

```bash
git stash drop stash@{0}
```

Delete a specific stash.

---

## Cherry Pick

### Apply Specific Commit

```bash
git cherry-pick <commit-id>
```

Copy a specific commit from another branch.

### Continue Cherry Pick

```bash
git cherry-pick --continue
```

Continue after resolving conflicts.

### Abort Cherry Pick

```bash
git cherry-pick --abort
```

Cancel cherry-pick operation.

### Skip Commit

```bash
git cherry-pick --skip
```

Skip the problematic commit.

---

## History Visualization

### Compact Log

```bash
git log --oneline
```

Show commit history in one line format.

### Graph View

```bash
git log --oneline --graph --all
```

Display branch and commit history visually.

### Show Commit Details

```bash
git show HEAD
```

View details of the latest commit.

---

## Conflict Resolution

### Check Status

```bash
git status
```

Display current repository status.

### Mark Conflict Resolved

```bash
git add <file-name>
```

Stage resolved files.

### Complete Merge or Cherry Pick

```bash
git commit -m "Resolve conflict"
```

Create commit after conflict resolution.
