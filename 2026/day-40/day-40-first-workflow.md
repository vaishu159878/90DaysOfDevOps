# Day 40 – My First GitHub Actions Workflow

## Objective

Create and run my first GitHub Actions workflow.

---

## Workflow File

```yaml
---
name: My First GitHub Actions Workflow

on:
  push:

jobs:
  greet:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Print Hello
        run: echo "Hello from GitHub Actions!"

      - name: Show Current Date and Time
        run: date

      - name: Show Branch Name
        run: echo "Branch: ${{ github.ref_name }}"

      - name: List Repository Files
        run: ls -la

      - name: Show Runner Operating System
        run: echo "Runner OS: $RUNNER_OS"
```