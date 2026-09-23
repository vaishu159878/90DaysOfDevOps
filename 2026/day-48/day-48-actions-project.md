# Day 48 – GitHub Actions Project: End-to-End CI/CD Pipeline

## Project Overview

This project is a complete end-to-end CI/CD capstone built using **GitHub Actions, Python Flask, Docker, Docker Hub, and AWS EC2**.

The pipeline combines reusable workflows, pull request validation, Docker image publishing, production deployment, and scheduled health checks.

### Project Repository

**GitHub:**  
https://github.com/vaishu159878/github-actions-capstone

### Docker Hub

**Docker Hub Repository:**  
https://hub.docker.com/r/vaishnavinalawade/github-actions-capstone

---

# 1. Pipeline Architecture

The project contains three main automation paths:

```text
PR opened / updated
        |
        v
+----------------------+
| Reusable Build/Test  |
+----------+-----------+
           |
           v
+----------------------+
|    PR Checks Pass    |
+----------------------+


Merge / Push to main
        |
        v
+----------------------+
| Reusable Build/Test  |
+----------+-----------+
           |
           v
+----------------------+
| Docker Build & Push  |
+----------+-----------+
           |
           +----------------------+
           |                      |
           v                      v
     latest tag             sha-<short-sha>
           |                      |
           +----------+-----------+
                      |
                      v
             +----------------+
             | Production     |
             | Deploy         |
             +----------------+


Every 12 hours / Manual
        |
        v
+----------------------+
| Pull latest image    |
+----------+-----------+
           |
           v
+----------------------+
| Run Docker container |
+----------+-----------+
           |
           v
+----------------------+
| curl /health         |
+----------+-----------+
           |
           v
+----------------------+
| GitHub Step Summary  |
+----------------------+
```

## Pipeline Flow

### Pull Request

```text
PR opened
   ↓
Build & Test
   ↓
PR Checks
```

No Docker image is built or pushed for the PR pipeline.

### Main Branch

```text
Push/Merge to main
   ↓
Build & Test
   ↓
Generate short SHA
   ↓
Docker Build & Push
   ├── latest
   └── sha-<7-character-sha>
   ↓
Production Deploy
```

### Scheduled Health Check

```text
Every 12 hours / Manual
   ↓
Pull latest Docker image
   ↓
Run container
   ↓
Wait 5 seconds
   ↓
Check /health
   ↓
Write GitHub Step Summary
   ↓
Stop and remove container
```

---

# 2. Project Structure

```text
github-actions-capstone/
│
├── .github/
│   └── workflows/
│       ├── reusable-build-test.yml
│       ├── reusable-docker.yml
│       ├── pr-pipeline.yml
│       ├── main-pipeline.yml
│       └── health-check.yml
│
├── app.py
├── requirements.txt
├── test_app.py
├── Dockerfile
├── README.md
└── day-48-actions-project.md
```

---

# 3. Reusable Build & Test Workflow

File:

```text
.github/workflows/reusable-build-test.yml
```

```yaml
name: Reusable Build and Test

on:
  workflow_call:
    inputs:
      python_version:
        required: false
        type: string
        default: "3.12"

      run_tests:
        required: false
        type: boolean
        default: true

    outputs:
      test_result:
        description: "Test result"
        value: ${{ jobs.build-test.outputs.test_result }}

jobs:
  build-test:
    runs-on: ubuntu-latest

    outputs:
      test_result: ${{ steps.test.outputs.result }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ inputs.python_version }}

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run tests
        id: test
        if: ${{ inputs.run_tests }}
        run: |
          if pytest; then
            echo "result=passed" >> "$GITHUB_OUTPUT"
          else
            echo "result=failed" >> "$GITHUB_OUTPUT"
            exit 1
          fi
```

### Purpose

This reusable workflow performs only build and test activities. It does not deploy or push Docker images.

---

# 4. Reusable Docker Build & Push Workflow

File:

```text
.github/workflows/reusable-docker.yml
```

```yaml
name: Reusable Docker Build and Push

on:
  workflow_call:
    inputs:
      image_name:
        required: true
        type: string

      tag:
        required: true
        type: string

    secrets:
      docker_username:
        required: true

      docker_token:
        required: true

    outputs:
      image_url:
        description: "Full Docker image URL"
        value: ${{ jobs.docker.outputs.image_url }}

jobs:
  docker:
    runs-on: ubuntu-latest

    outputs:
      image_url: ${{ steps.image.outputs.image_url }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.docker_username }}
          password: ${{ secrets.docker_token }}

      - name: Build and push image
        run: |
          docker build \
            -t ${{ inputs.image_name }}:${{ inputs.tag }} .

          docker push \
            ${{ inputs.image_name }}:${{ inputs.tag }}

      - name: Set image output
        id: image
        run: |
          echo "image_url=${{ inputs.image_name }}:${{ inputs.tag }}" >> "$GITHUB_OUTPUT"
```

### Purpose

This reusable workflow builds the Docker image and pushes it to Docker Hub using secure GitHub repository secrets.

---

# 5. Pull Request Pipeline

File:

```text
.github/workflows/pr-pipeline.yml
```

```yaml
name: PR Pipeline

on:
  pull_request:
    branches:
      - main
    types:
      - opened
      - synchronize

jobs:
  build-test:
    uses: ./.github/workflows/reusable-build-test.yml
    with:
      python_version: "3.12"
      run_tests: true

  pr-comment:
    needs: build-test
    runs-on: ubuntu-latest

    steps:
      - name: PR checks summary
        run: |
          echo "PR checks passed for branch: ${{ github.head_ref }}"
```

### PR Verification

The PR pipeline successfully ran:

```text
build-test   ✅
pr-comment   ✅
```

There was no Docker build/push job in the PR workflow.


# 6. Main Branch Pipeline

File:

```text
.github/workflows/main-pipeline.yml
```

```yaml
name: Main Pipeline

on:
  push:
    branches:
      - main

jobs:
  build-test:
    uses: ./.github/workflows/reusable-build-test.yml
    with:
      python_version: "3.12"
      run_tests: true

  docker-latest:
    needs: build-test
    uses: ./.github/workflows/reusable-docker.yml
    with:
      image_name: vaishnavinalawade/github-actions-capstone
      tag: latest
    secrets:
      docker_username: ${{ secrets.DOCKER_USERNAME }}
      docker_token: ${{ secrets.DOCKER_TOKEN }}

  prepare:
    needs: build-test
    runs-on: ubuntu-latest

    outputs:
      short_sha: ${{ steps.sha.outputs.short_sha }}

    steps:
      - name: Generate short SHA
        id: sha
        run: |
          echo "short_sha=$(echo '${{ github.sha }}' | cut -c1-7)" >> "$GITHUB_OUTPUT"

  docker-sha:
    needs: prepare
    uses: ./.github/workflows/reusable-docker.yml
    with:
      image_name: vaishnavinalawade/github-actions-capstone
      tag: sha-${{ needs.prepare.outputs.short_sha }}
    secrets:
      docker_username: ${{ secrets.DOCKER_USERNAME }}
      docker_token: ${{ secrets.DOCKER_TOKEN }}

  deploy:
    needs:
      - docker-latest
      - docker-sha

    runs-on: ubuntu-latest

    environment:
      name: production

    steps:
      - name: Deploy
        run: |
          echo "Deploying image: ${{ needs.docker-latest.outputs.image_url }} to production"
```

### Main Pipeline Verification

The main branch pipeline successfully executed:

```text
build-test      ✅
docker-latest   ✅
docker-sha      ✅
deploy          ✅
```

The workflow completed successfully.


# 7. Scheduled Docker Health Check

File:

```text
.github/workflows/health-check.yml
```

```yaml
name: Docker Health Check

on:
  schedule:
    - cron: "0 */12 * * *"

  workflow_dispatch:

jobs:
  health-check:
    runs-on: ubuntu-latest

    steps:
      - name: Pull latest Docker image
        run: |
          docker pull vaishnavinalawade/github-actions-capstone:latest

      - name: Run container
        run: |
          docker run -d \
            --name health-check-container \
            -p 8000:5000 \
            vaishnavinalawade/github-actions-capstone:latest

      - name: Wait for application
        run: sleep 5

      - name: Check health endpoint
        run: |
          if curl --fail http://localhost:8000/health; then
            echo "Health check PASSED"
          else
            echo "Health check FAILED"
            exit 1
          fi

      - name: Create health summary
        if: always()
        run: |
          echo "## Health Check Report" >> "$GITHUB_STEP_SUMMARY"
          echo "- Image: vaishnavinalawade/github-actions-capstone:latest" >> "$GITHUB_STEP_SUMMARY"

          if curl --fail http://localhost:8000/health > /dev/null 2>&1; then
            echo "- Status: PASSED" >> "$GITHUB_STEP_SUMMARY"
          else
            echo "- Status: FAILED" >> "$GITHUB_STEP_SUMMARY"
          fi

          echo "- Time: $(date)" >> "$GITHUB_STEP_SUMMARY"

      - name: Stop and remove container
        if: always()
        run: |
          docker stop health-check-container || true
          docker rm health-check-container || true
```

### Health Check Verification

The manual health-check run completed successfully:

```text
Pull latest Docker image    ✅
Run container               ✅
Wait for application        ✅
Check health endpoint       ✅
Create health summary      ✅
```


# 8. Docker Hub

Docker Hub repository:

**https://hub.docker.com/r/vaishnavinalawade/github-actions-capstone**

The pipeline successfully pushed Docker images to Docker Hub.

### Docker Image

```text
vaishnavinalawade/github-actions-capstone:latest
```

### Short SHA Image

The correctly formatted short-SHA tag generated by the pipeline is:

```text
vaishnavinalawade/github-actions-capstone:sha-303ffda
```

---

# 9. GitHub Secrets

The Docker workflow uses repository secrets rather than hard-coded credentials.

Configured secrets:

```text
DOCKER_USERNAME
DOCKER_TOKEN
```

The Docker Hub access token is used for authentication.

---

# 10. Production Environment

The main pipeline uses the GitHub Actions environment:

```text
production
```

The deployment job depends on the Docker build jobs:

```text
build-test
    ↓
docker-latest
    ↓
docker-sha
    ↓
deploy
```

The environment can also be configured with required reviewers for manual deployment approval.

---



# 11. What I Would Improve Next

The current pipeline demonstrates the core CI/CD flow. The next improvements I would add are:

### 1. DevSecOps Security Scanning

Add **Trivy** after the Docker image is built to scan for vulnerabilities.

```text
Build Image
    ↓
Trivy Scan
    ↓
Fail on CRITICAL vulnerabilities
    ↓
Push Image
```

### 2. Multi-Environment Deployment

Introduce:

```text
Development
     ↓
Staging
     ↓
Production
```

with environment-specific configuration and approvals.

### 3. Automatic Rollback

If a deployment fails its health check, automatically roll back to the previous known-good image.

### 4. Notifications

Add Slack or another notification system for:

- Failed builds
- Failed deployments
- Successful production deployments
- Health-check failures

### 5. AWS ECR

Instead of Docker Hub, integrate AWS ECR for container image storage and connect the deployment workflow with AWS infrastructure.

### 6. Kubernetes Deployment

Extend the pipeline to deploy the application to Kubernetes/EKS.

```text
GitHub
   ↓
GitHub Actions
   ↓
Docker Build
   ↓
Container Registry
   ↓
EKS
   ↓
Kubernetes Deployment
```

### 7. Observability

Add:

- Prometheus
- Grafana
- Container metrics
- Application metrics
- Alerting

---


# 12. Final Pipeline

```text
                    GitHub Repository
                           |
              +------------+------------+
              |                         |
         Pull Request               Push to main
              |                         |
              v                         v
       Build & Test              Build & Test
              |                         |
              v                         v
        PR Checks                Docker Build
                                      |
                         +------------+------------+
                         |                         |
                         v                         v
                       latest                 sha-303ffda
                         |                         |
                         +------------+------------+
                                      |
                                      v
                              Production Deploy

                    Every 12 hours / Manual
                              |
                              v
                         Health Check
                              |
                              v
                           /health
                              |
                              v
                           PASSED
```

---

## Conclusion

This capstone connected the GitHub Actions concepts into one complete CI/CD workflow:

**Code → Pull Request → Automated Tests → Docker Build → Docker Hub → Production Deployment → Scheduled Health Check**

The project demonstrates how reusable GitHub Actions workflows can be combined with Docker and automated testing to create a practical CI/CD pipeline.
