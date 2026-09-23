# Reusable Workflows & Composite Actions

## Overview

While practicing CI/CD with GitHub Actions, I started noticing a
problem:

> Why repeat the same workflow logic when it can be reused?

This led me to explore two powerful GitHub Actions concepts:

-   Reusable Workflows
-   Composite Actions

I built both from scratch and connected them to a working GitHub Actions
pipeline.

------------------------------------------------------------------------

## 1. What is a Reusable Workflow?

A reusable workflow is a GitHub Actions workflow that can be called by
another workflow.

It helps avoid duplicating the same CI/CD logic across multiple
workflows or repositories.

A reusable workflow uses the `workflow_call` trigger.

``` yaml
on:
  workflow_call:
```

A reusable workflow can define:

-   Inputs
-   Secrets
-   Outputs
-   Jobs
-   Multiple steps

### Where does it live?

Reusable workflows must be stored inside:

``` text
.github/workflows/
```

------------------------------------------------------------------------

## 2. What is `workflow_call`?

`workflow_call` allows one GitHub Actions workflow to call another
workflow.

For example:

``` yaml
on:
  workflow_call:
```

The caller workflow can pass inputs and secrets to the reusable
workflow.

------------------------------------------------------------------------

## 3. Reusable Workflow vs Regular Action

A reusable workflow is called at the **job level**:

``` yaml
jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
```

An action is normally called inside a **step**:

``` yaml
steps:
  - uses: actions/checkout@v4
```

In simple terms:

``` text
Reusable Workflow → Job level
Action            → Step level
```

------------------------------------------------------------------------

# 4. Reusable Workflow

File:

``` text
.github/workflows/reusable-build.yml
```

``` yaml
name: Reusable Build

on:
  workflow_call:
    inputs:
      app_name:
        description: "Application name"
        required: true
        type: string

      environment:
        description: "Deployment environment"
        required: true
        default: staging
        type: string

    secrets:
      docker_token:
        description: "Docker token"
        required: true

    outputs:
      build_version:
        description: "Generated build version"
        value: ${{ jobs.build.outputs.build_version }}

jobs:
  build:
    runs-on: ubuntu-latest

    outputs:
      build_version: ${{ steps.version.outputs.build_version }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Print build information
        run: |
          echo "Building ${{ inputs.app_name }} for ${{ inputs.environment }}"

      - name: Check Docker token
        run: |
          if [ -n "${{ secrets.docker_token }}" ]; then
            echo "Docker token is set: true"
          else
            echo "Docker token is set: false"
          fi

      - name: Generate build version
        id: version
        run: |
          SHORT_SHA=$(git rev-parse --short HEAD)
          VERSION="v1.0-${SHORT_SHA}"

          echo "Build version: $VERSION"
          echo "build_version=$VERSION" >> "$GITHUB_OUTPUT"
```

------------------------------------------------------------------------

# 5. Caller Workflow

File:

``` text
.github/workflows/call-build.yml
```

``` yaml
name: Call Reusable Build

on:
  push:
    branches:
      - main

jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml

    with:
      app_name: "my-web-app"
      environment: "production"

    secrets:
      docker_token: ${{ secrets.DOCKER_TOKEN }}

  show-version:
    needs: build
    runs-on: ubuntu-latest

    steps:
      - name: Print build version
        run: |
          echo "Build version from reusable workflow:"
          echo "${{ needs.build.outputs.build_version }}"
```

------------------------------------------------------------------------

# 6. Workflow Flow

The complete flow was:

``` text
Git Push
   ↓
Caller Workflow
   ↓
Reusable Workflow
   ↓
Build Application
   ↓
Generate Build Version
   ↓
Pass Output to Next Job
```

The workflow successfully generated:

``` text
v1.0-fd94181
```

and passed the value to the next job.

------------------------------------------------------------------------

# 7. Secrets

The reusable workflow receives a Docker token:

``` yaml
secrets:
  docker_token:
    required: true
```

The caller passes the repository secret:

``` yaml
secrets:
  docker_token: ${{ secrets.DOCKER_TOKEN }}
```

The secret itself is never printed.

Instead, the workflow checks whether it is set:

``` text
Docker token is set: true
```

This keeps the actual secret value hidden.

------------------------------------------------------------------------

# 8. Composite Action

I also created a custom composite action.

File:

``` text
.github/actions/setup-and-greet/action.yml
```

``` yaml
name: Setup and Greet
description: "Custom composite action that greets the user and displays runner information"

inputs:
  name:
    description: "Name to greet"
    required: true

  language:
    description: "Greeting language"
    required: false
    default: "en"

outputs:
  greeted:
    description: "Whether the greeting was completed"
    value: ${{ steps.greeting.outputs.greeted }}

runs:
  using: "composite"

  steps:
    - name: Print greeting
      id: greeting
      shell: bash
      run: |
        case "${{ inputs.language }}" in
          en)
            echo "Hello, ${{ inputs.name }}!"
            ;;
          hi)
            echo "Namaste, ${{ inputs.name }}!"
            ;;
          mr)
            echo "Namaskar, ${{ inputs.name }}!"
            ;;
          *)
            echo "Hello, ${{ inputs.name }}!"
            ;;
        esac

        echo "greeted=true" >> "$GITHUB_OUTPUT"

    - name: Print runner information
      shell: bash
      run: |
        echo "Current date: $(date)"
        echo "Runner OS: $RUNNER_OS"
```

------------------------------------------------------------------------

# 9. Composite Action Workflow

File:

``` text
.github/workflows/composite-test.yml
```

``` yaml
name: Test Composite Action

on:
  push:
    branches:
      - main

jobs:
  test-action:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Run custom greeting action
        id: greet
        uses: ./.github/actions/setup-and-greet
        with:
          name: "Vaishnavi"
          language: "en"

      - name: Check action output
        run: |
          echo "Greeting completed: ${{ steps.greet.outputs.greeted }}"
```

------------------------------------------------------------------------

# 10. Composite Action Flow

``` text
Input Name
   ↓
Custom Greeting
   ↓
Runner Information
   ↓
Current Date
   ↓
Action Output
```

The action successfully produced:

``` text
Hello, Vaishnavi!
Current date: ...
Runner OS: Linux
Greeting completed: true
```

------------------------------------------------------------------------

# 11. Reusable Workflow vs Composite Action

  ----------------------------------------------------------------------------------
                          Reusable Workflow       Composite Action
  ----------------------- ----------------------- ----------------------------------
  Triggered by            `workflow_call`         `uses:` in a step

  Can contain jobs?       Yes                     No

  Can contain multiple    Yes                     Yes
  steps?                                          

  Lives where?            `.github/workflows/`    `.github/actions/<action-name>/`

  Can accept secrets      Yes                     Not directly as workflow secrets
  directly?                                       

  Best for                Reusing complete CI/CD  Reusing groups of steps
                          workflows               
  ----------------------------------------------------------------------------------

------------------------------------------------------------------------

# 12. Key Difference

### Reusable Workflow

``` text
Workflow
   ↓
Job
   ↓
Reusable Workflow
   ↓
Jobs / Steps
```

### Composite Action

``` text
Workflow
   ↓
Job
   ↓
Step
   ↓
Composite Action
   ↓
Multiple Steps
```

------------------------------------------------------------------------

# 13. Verification

The reusable workflow completed successfully.

The build job printed:

``` text
Building my-web-app for production
Docker token is set: true
Build version: v1.0-fd94181
```

The second job successfully received the generated build version.

The composite action successfully printed:

``` text
Hello, Vaishnavi!
Runner OS: Linux
Greeting completed: true
```

------------------------------------------------------------------------

