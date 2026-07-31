# Day 39 – CI/CD Concepts

## What is CI/CD?

CI/CD (Continuous Integration and Continuous Delivery/Deployment) is a DevOps practice that automates the process of building, testing, and deploying software. It helps teams release software faster, with fewer errors and more confidence.

---

# Task 1: The Problem

## 1. What can go wrong when 5 developers manually deploy to production?

- Developers may overwrite each other's changes.
- Manual deployment can lead to human errors.
- Different environments may cause unexpected issues.
- Bugs may reach production because testing is skipped.
- Rollbacks become difficult if something breaks.
- Deployments take longer and require coordination.

---

## 2. What does "It works on my machine" mean?

"It works on my machine" means the application runs correctly on the developer's computer but fails in another environment (testing or production).

### Why is it a real problem?

- Different operating systems
- Different software versions
- Missing dependencies
- Different environment variables
- Configuration differences

CI/CD solves this by running builds and tests in a consistent environment.

---

## 3. How many times a day can a team safely deploy manually?

Usually only **1–3 deployments per day**, depending on the project and team size.

Frequent manual deployments increase the chances of mistakes.

With CI/CD, teams can safely deploy many times a day because testing and deployment are automated.

---

# Task 2: CI vs CD

## Continuous Integration (CI)

Continuous Integration is the practice of frequently merging code changes into a shared repository.

Every code push automatically triggers building and testing to detect problems early.

### Example

A developer pushes code to GitHub.
GitHub Actions automatically builds the project and runs unit tests.

---

## Continuous Delivery (CD)

Continuous Delivery automatically prepares the application for deployment after all tests pass.

The deployment to production still requires **manual approval**.

### Example

After successful tests, a Docker image is created and uploaded to Docker Hub.
A release manager clicks **Deploy** to production.

---

## Continuous Deployment (CD)

Continuous Deployment automatically deploys every successful build to production without manual approval.

Only fully tested code reaches production.

### Example

A website automatically updates after every successful commit to the main branch.

---

# CI vs Continuous Delivery vs Continuous Deployment

| Feature            | Continuous Integration  | Continuous Delivery | Continuous Deployment |
|--------------------|-------------------------|---------------------|-----------------------|
| Build              | ✅                      | ✅                 | ✅                    |
| Automated Testing  | ✅                      | ✅                 | ✅                    |
| Deployment         | ❌                      | Manual Approval     | Fully Automatic       |
| Human Approval     | Not Needed               | Required           | Not Required           |

---

# Task 3: Pipeline Anatomy

## Trigger

An event that starts the pipeline.

Examples:
- Push
- Pull Request
- Schedule
- Manual trigger

---

## Stage

A logical phase in the pipeline.

Examples:
- Build
- Test
- Deploy

---

## Job

A collection of related steps that run on the same runner.

Example:
Build Job
- Install dependencies
- Compile application
- Build Docker image

---

## Step

A single command or action inside a job.

Examples:

```bash
npm install
```

```bash
docker build -t myapp .
```

---

## Runner

A machine (virtual or physical) that executes pipeline jobs.

Examples:

- GitHub-hosted Runner
- Self-hosted Runner

---

## Artifact

A file or package produced by a job that can be used later.

Examples:

- JAR file
- ZIP package
- Docker image
- Test reports

---

# Task 4: CI/CD Pipeline Diagram

```
          Developer
              │
              │ Push Code
              ▼
      GitHub Repository
              │
              ▼
        Trigger Pipeline
              │
              ▼
      ┌─────────────────┐
      │ Build Stage     │
      │ Compile Project │
      └─────────────────┘
              │
              ▼
      ┌─────────────────┐
      │ Test Stage      │
      │ Run Unit Tests  │
      └─────────────────┘
              │
              ▼
      ┌─────────────────────┐
      │ Docker Build Stage  │
      │ Build Docker Image  │
      └─────────────────────┘
              │
              ▼
      ┌─────────────────┐
      │ Deploy Stage    │
      │ Staging Server  │
      └─────────────────┘
              │
              ▼
        Application Ready
```

---

# Task 5: Explore in the Wild

## Repository Chosen

React

Repository:
https://github.com/facebook/react

Workflow File:

`.github/workflows/runtime_build_and_test.yml`

### What triggers it?

- Push
- Pull Request

### How many jobs does it have?

Multiple jobs (build, test, lint and other validation jobs).

### What does it do?

- Installs dependencies
- Builds the project
- Runs tests
- Performs linting
- Verifies code quality before merging

---
