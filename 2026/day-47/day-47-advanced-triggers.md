# Advanced GitHub Actions Triggers & Event-Driven CI/CD

## Overview

This practice focused on understanding how GitHub Actions can respond to different events instead of relying only on `push`.

I built and tested workflows for:

- Pull Request lifecycle events
- Automated PR validation
- Scheduled workflows with cron
- Manual workflow execution
- Path and branch filters
- Workflow chaining using `workflow_run`
- External event triggers using `repository_dispatch`

The goal was to understand how event-driven automation can be used to build practical CI/CD pipelines.

---

## Repository Structure

```text
.github/
└── workflows/
    ├── pr-lifecycle.yml
    ├── pr-checks.yml
    ├── scheduled-tasks.yml
    ├── smart-triggers.yml
    ├── smart-triggers-ignore.yml
    ├── tests.yml
    ├── deploy-after-tests.yml
    └── external-trigger.yml

2026/
└── day-47/
    └── day-47-advanced-triggers.md
```

---

# 1. Pull Request Lifecycle

### Workflow

```text
.github/workflows/pr-lifecycle.yml
```

The workflow listens for these Pull Request events:

```yaml
types:
  - opened
  - synchronize
  - reopened
  - closed
```

It displays:

- Event type
- Pull Request title
- Pull Request author
- Source branch
- Target branch

It also detects whether a closed PR was actually merged.

```yaml
if: github.event.action == 'closed' &&
    github.event.pull_request.merged == true
```

### Tested Events

```text
opened       ✅
synchronize  ✅
reopened     ✅
closed       ✅
merged=true  ✅
```

---

# 2. Pull Request Validation

### Workflow

```text
.github/workflows/pr-checks.yml
```

The workflow contains three jobs.

## File Size Check

Checks for files larger than 1 MB.

```text
File <= 1 MB     → Pass
File > 1 MB      → Fail
```

Tested with a 2 MB file.

## Branch Name Check

Allowed branch patterns:

```text
feature/*
fix/*
docs/*
```

Example of a valid branch:

```text
feature/pr-validation-test
```

Example of an invalid branch:

```text
testing-bad-branch
```

## PR Description Check

Checks whether the Pull Request description is empty.

An empty description produces a warning instead of failing the workflow.

---

# 3. Scheduled Workflows

### Workflow

```text
.github/workflows/scheduled-tasks.yml
```

Two cron schedules were configured.

### Every Monday at 2:30 AM UTC

```text
30 2 * * 1
```

### Every 6 hours

```text
0 */6 * * *
```

The workflow also supports:

```yaml
workflow_dispatch:
```

This allows the workflow to be tested manually without waiting for the scheduled time.

### Health Check

The workflow uses `curl` to check GitHub:

```bash
response=$(curl -s -o /dev/null -w "%{http_code}" https://github.com)
```

The test returned:

```text
HTTP Response Code: 200
Health check passed.
```

---

# 4. Additional Cron Expressions

## Every weekday at 9 AM IST

GitHub Actions cron uses UTC.

9:00 AM IST = 3:30 AM UTC.

```text
30 3 * * 1-5
```

## First day of every month at midnight UTC

```text
0 0 1 * *
```

---

# 5. Path and Branch Filters

### Workflows

```text
.github/workflows/smart-triggers.yml
.github/workflows/smart-triggers-ignore.yml
```

## paths

The workflow runs only when files under these directories change:

```yaml
paths:
  - 'src/**'
  - 'app/**'
```

For example:

```text
src/app.js → workflow runs
app/index.html → workflow runs
README.md → workflow skipped
```

## paths-ignore

The second workflow ignores:

```yaml
paths-ignore:
  - '*.md'
  - 'docs/**'
```

This is useful when documentation-only changes should not trigger certain workflows.

---

# 6. workflow_run

### Workflows

```text
.github/workflows/tests.yml
.github/workflows/deploy-after-tests.yml
```

The workflow chain is:

```text
Push
  ↓
Run Tests
  ↓
workflow_run
  ↓
Deploy After Tests
```

The deployment workflow checks:

```yaml
github.event.workflow_run.conclusion == 'success'
```

Deployment proceeds only when the test workflow succeeds.

### Tested Result

```text
Run Tests              → ✅
workflow_run triggered → ✅
Deploy After Tests     → ✅
Deployment             → ✅
```

---

# 7. workflow_run vs workflow_call

## workflow_run

`workflow_run` allows one workflow to react to the completion of another workflow.

Example:

```text
Run Tests
    ↓
workflow_run
    ↓
Deploy
```

## workflow_call

`workflow_call` is used to create reusable workflows that can be called by another workflow.

Example:

```text
Caller Workflow
      ↓
Reusable Workflow
```

### Difference

```text
workflow_run
→ Event-driven workflow chaining

workflow_call
→ Reusable workflow logic
```

---

# 8. repository_dispatch

### Workflow

```text
.github/workflows/external-trigger.yml
```

The workflow listens for:

```yaml
repository_dispatch:
  types:
    - deploy-request
```

An external event can send data through:

```yaml
github.event.client_payload.environment
```

Example payload:

```json
{
  "environment": "production"
}
```

### Tested Result

```text
External deployment request received.
Environment: production

Starting deployment...
Deployment environment:
production
```

---

# 9. Event-Driven CI/CD Flow

The different workflows demonstrate several ways GitHub Actions can respond to events.

```text
                     GitHub Actions
                           |
       +-------------------+-------------------+
       |                   |                   |
       ↓                   ↓                   ↓
 Pull Request          Schedule           External Event
       |                   |                   |
       ↓                   ↓                   ↓
 PR Validation       Health Check       repository_dispatch
       |
       ↓
     Merge
       |
       ↓
    Deployment
```

Another workflow demonstrates:

```text
Push
 ↓
Run Tests
 ↓
workflow_run
 ↓
Deploy
```

---


