# Day 41 – Triggers & Matrix Builds

Today I learned about different ways to trigger GitHub Actions workflows and also practiced matrix builds.

The main things I practiced today were:

- Pull Request trigger
- Scheduled trigger
- Manual trigger
- Workflow inputs
- Matrix builds
- Multiple operating systems
- Matrix exclude
- `fail-fast`

---

## Task 1 – Pull Request Trigger

I created:

`.github/workflows/pr-check.yml`

```yaml
name: PR Check

on:
  pull_request:
    branches:
      - main
    types:
      - opened
      - synchronize

jobs:
  pr-check:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Print PR branch
        run: |
          echo "PR check running for branch: ${{ github.head_ref }}"
```

### What I did

I created a new branch:

```text
day-41-pr-test
```

Then I pushed a commit and created a Pull Request against `main`.

The workflow automatically started when the PR was opened.

When I pushed another commit to the same PR, the workflow also ran again because I used the `synchronize` event.

### Output

```text
PR check running for branch: day-41-pr-test
```

### What I learned

A Pull Request workflow can be used to run checks before changes are merged into the main branch.

---

## Task 2 – Scheduled Trigger

I created:

`.github/workflows/scheduled.yml`

```yaml
name: Scheduled Workflow

on:
  schedule:
    - cron: '0 0 * * *'

jobs:
  scheduled-job:
    runs-on: ubuntu-latest

    steps:
      - name: Show schedule message
        run: |
          echo "Scheduled workflow is running."
          echo "Current UTC time:"
          date -u
```

The cron expression:

```text
0 0 * * *
```

means the workflow runs every day at midnight UTC.

### Monday at 9 AM

The cron expression for every Monday at 9 AM UTC is:

```text
0 9 * * 1
```

Cron format:

```text
minute hour day-of-month month day-of-week
```

---

## Task 3 – Manual Trigger

I created:

`.github/workflows/manual.yml`

```yaml
name: Manual Deployment

on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Enter environment name"
        required: true
        default: "staging"
        type: choice
        options:
          - staging
          - production

jobs:
  manual-job:
    runs-on: ubuntu-latest

    steps:
      - name: Show selected environment
        run: |
          echo "Selected environment: ${{ inputs.environment }}"
```

### What I did

I went to:

```text
Actions → Manual Deployment → Run workflow
```

I tested the two environment options:

```text
staging
production
```

The workflow printed the selected environment.

Example:

```text
Selected environment: staging
```

This helped me understand how `workflow_dispatch` can be used to manually run a workflow and accept input from the user.

---

## Task 4 – Matrix Builds

I created:

`.github/workflows/matrix.yml`

First, I tested three Python versions:

```yaml
strategy:
  matrix:
    python-version:
      - "3.10"
      - "3.11"
      - "3.12"
```

This created three jobs:

```text
Python 3.10
Python 3.11
Python 3.12
```

All three jobs ran successfully.

---

## Matrix with Operating Systems

Then I added two operating systems:

```yaml
os:
  - ubuntu-latest
  - windows-latest
```

Now GitHub Actions created combinations of Python versions and operating systems.

### Calculation

```text
3 Python versions × 2 operating systems = 6 jobs
```

The six combinations were:

```text
Python 3.10 + Ubuntu
Python 3.10 + Windows

Python 3.11 + Ubuntu
Python 3.11 + Windows

Python 3.12 + Ubuntu
Python 3.12 + Windows
```

I could see the jobs running separately in GitHub Actions.

---

## Task 5 – Exclude

Next I excluded this combination:

```yaml
exclude:
  - os: windows-latest
    python-version: "3.10"
```

Before excluding:

```text
3 × 2 = 6 jobs
```

After excluding one combination:

```text
6 - 1 = 5 jobs
```

The five jobs were:

```text
Python 3.10 + Ubuntu
Python 3.11 + Ubuntu
Python 3.11 + Windows
Python 3.12 + Ubuntu
Python 3.12 + Windows
```

I verified this in the GitHub Actions run.

---

## fail-fast

I added:

```yaml
strategy:
  fail-fast: false
```

Then I intentionally made one matrix combination fail:

```yaml
- name: Intentional failure
  if: matrix.python-version == '3.11' && matrix.os == 'ubuntu-latest'
  run: |
    echo "This job is intentionally failing"
    exit 1
```

The result was:

```text
Python 3.10 + Ubuntu     ✅
Python 3.11 + Ubuntu     ❌
Python 3.11 + Windows    ✅
Python 3.12 + Ubuntu     ✅
Python 3.12 + Windows    ✅
```

The overall workflow showed failure because one job failed, but the other matrix jobs continued running.

This helped me understand how `fail-fast: false` works.

---

## fail-fast: true vs false

### `fail-fast: true`

This is the default.

If a matrix job fails, GitHub Actions can cancel other matrix jobs that are still running.

### `fail-fast: false`

Other matrix jobs continue running even if one matrix job fails.

I tested this using an intentional failure in the Python 3.11 + Ubuntu combination.

---

# What I Learned Today

Today I learned that GitHub Actions workflows can be triggered in different ways.

The main triggers I practiced were:

```text
push
pull_request
schedule
workflow_dispatch
```

I also learned how matrix builds make it easier to test the same job across different versions and operating systems.

The main concepts I practiced were:

```text
Matrix
Exclude
fail-fast
workflow_dispatch
Inputs
Cron
Pull Request triggers
```

The most useful thing I learned today was that I don't need to create separate jobs manually for every environment. A matrix can create those combinations automatically.

---

