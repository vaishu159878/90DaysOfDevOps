# Day 43 – Jobs, Steps, Env Vars & Conditionals

Today I practiced controlling the flow of GitHub Actions workflows.

Instead of having only a single job running step by step, I worked with multiple jobs, dependencies, environment variables, job outputs, and conditions.

---

## What I Practiced

- Multi-job workflows
- Job dependencies using `needs`
- Environment variables
- GitHub context variables
- Passing values between jobs using outputs
- Conditional steps
- `failure()`
- `continue-on-error`
- Push-only jobs
- Parallel jobs
- Smart pipeline

---

# 1. Multi-Job Workflow

I created a workflow with three jobs:

```text
build
  ↓
test
  ↓
deploy
```

The workflow contains:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build application
        run: echo "Building the app"

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Run tests
        run: echo "Running tests"

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy application
        run: echo "Deploying"
```

### What `needs:` does

I used `needs:` to create a dependency between jobs.

For example:

```yaml
test:
  needs: build
```

This means the `test` job waits for the `build` job to complete successfully.

Similarly:

```yaml
deploy:
  needs: test
```

means the `deploy` job waits for `test`.

So the dependency chain is:

```text
build
  ↓
test
  ↓
deploy
```

I also checked the workflow graph in the GitHub Actions tab.

---

# 2. Environment Variables

I practiced environment variables at three different levels.

## Workflow Level

```yaml
env:
  APP_NAME: myapp
```

This variable is available throughout the workflow.

## Job Level

```yaml
env:
  ENVIRONMENT: staging
```

This variable is available to the steps inside that job.

## Step Level

```yaml
env:
  VERSION: 1.0.0
```

This variable is available only inside that step.

I printed all three values:

```yaml
run: |
  echo "Application: $APP_NAME"
  echo "Environment: $ENVIRONMENT"
  echo "Version: $VERSION"
```

---

# 3. GitHub Context Variables

I also practiced GitHub context variables.

For the commit SHA:

```yaml
echo "Commit SHA: ${{ github.sha }}"
```

For the user who triggered the workflow:

```yaml
echo "Triggered by: ${{ github.actor }}"
```

These values are provided by GitHub Actions.

---

# 4. Job Outputs

I practiced passing a value from one job to another.

First, I generated today's date:

```yaml
- name: Generate today's date
  id: date
  run: |
    echo "today=$(date)" >> "$GITHUB_OUTPUT"
```

Then I exposed it as a job output:

```yaml
outputs:
  today: ${{ steps.date.outputs.today }}
```

The next job can access the value using:

```yaml
${{ needs.generate-date.outputs.today }}
```

The flow is:

```text
generate-date
      |
      | today
      ↓
display-date
```

### Why use job outputs?

Job outputs are useful when one job generates some information that another job needs.

For example:

- Version number
- Docker image tag
- Deployment ID
- Environment name
- Generated date

---

# 5. Conditionals

I practiced running steps only when specific conditions are true.

## Run only on main branch

```yaml
if: github.ref == 'refs/heads/main'
```

This step runs only when the workflow is running on the `main` branch.

---

## Run after a failure

I used:

```yaml
if: failure()
```

This allows a step to run when a previous step has failed.

Example:

```yaml
- name: Run after failure
  if: failure()
  run: echo "The previous step failed"
```

---

# 6. Continue on Error

I also practiced:

```yaml
continue-on-error: true
```

Example:

```yaml
- name: Test something
  continue-on-error: true
  run: |
    echo "This step is allowed to fail"
    exit 1
```

Normally, a failed step can stop the job.

With:

```yaml
continue-on-error: true
```

the step can fail but the workflow continues with the next steps.

---

# 7. Push-Only Job

I created a job that runs only for push events.

```yaml
push-only-job:
  if: github.event_name == 'push'
  runs-on: ubuntu-latest

  steps:
    - name: Push event
      run: echo "This job runs only for push events"
```

The condition checks the GitHub event:

```yaml
github.event_name == 'push'
```

---

# 8. Smart Pipeline

I created a workflow called `smart-pipeline.yml`.

It contains:

```text
        ┌─────────┐
        │  lint   │
        └────┬────┘
             │
             ▼
        ┌─────────┐
        │ summary │
        └─────────┘
             ▲
             │
        ┌────┴────┐
        │  test   │
        └─────────┘
```

The `lint` and `test` jobs can run in parallel because they don't depend on each other.

The `summary` job waits for both:

```yaml
needs: [lint, test]
```

The summary job prints whether the workflow was triggered from:

```text
main branch
```

or:

```text
feature branch
```

It also prints the commit message.

---


