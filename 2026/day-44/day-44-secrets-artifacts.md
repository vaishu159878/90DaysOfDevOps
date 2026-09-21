# Day 44 – Secrets, Artifacts & Real Tests in CI

## Overview

Today I worked on making my GitHub Actions pipeline do more real work.

I practiced working with **GitHub Secrets, environment variables, artifacts, multiple jobs, real tests, and caching**.

The main goal was to understand how these things are used in a real CI pipeline.

---

## What I Practiced

### 1. GitHub Secrets

I created repository secrets in:

**GitHub → Settings → Secrets and variables → Actions**

Secrets created:

- `MY_SECRET_MESSAGE`
- `DOCKER_USERNAME`
- `DOCKER_TOKEN`

I used the secret through an environment variable instead of hardcoding the value.

The workflow checks whether the secret is available without displaying the actual secret.

Example output:

```text
The secret is set: true
```

I also tested printing the secret to understand how GitHub masks secret values in workflow logs.

### Why should secrets not be printed?

Secrets can contain passwords, tokens, API keys, and other sensitive information.

Even though GitHub automatically masks many secret values in logs, secrets should never be intentionally printed because exposing credentials can create security risks.

---

## 2. Using Secrets as Environment Variables

I passed the GitHub secret to a workflow step using an environment variable.

Example:

```yaml
env:
  MY_SECRET: ${{ secrets.MY_SECRET_MESSAGE }}
```

This allows the workflow to use the secret without hardcoding it inside the workflow file.

---

## 3. Uploading Artifacts

I created a test report during the workflow and uploaded it using:

```yaml
uses: actions/upload-artifact@v4
```

The generated artifact was:

```text
test-report
```

After the workflow completed, I downloaded the artifact from the GitHub Actions run.

---

## 4. Sharing Artifacts Between Jobs

I created two jobs.

### Job 1

- Generated a file
- Uploaded it as an artifact

### Job 2

- Downloaded the artifact
- Read and printed its contents

I used:

```yaml
needs: create-artifact
```

to make Job 2 wait for Job 1.

This helped me understand how artifacts can be used to pass build outputs between different jobs.

### Where can artifacts be useful?

Artifacts can be useful for storing:

- Test reports
- Build files
- Logs
- Screenshots
- Deployment packages
- Debugging information

---

## 5. Running Real Tests in CI

I created a shell script:

```text
test_script.sh
```

The script contains simple tests and exits with a non-zero status when a test fails.

I configured GitHub Actions to run the script automatically.

The workflow:

1. Checks out the repository
2. Makes the script executable
3. Runs the test
4. Fails if the script exits with a non-zero status

Passing output:

```text
Running Day 44 test...
Test 1: PASS
Test 2: PASS
All tests passed!
```

---

## 6. Testing a Failed Pipeline

I intentionally changed the test so that it would fail.

The GitHub Actions workflow became red and showed:

```text
Test 1: FAIL
```

The workflow exited with:

```text
exit code 1
```

After fixing the script, I pushed the changes again and the workflow became green.

This helped me understand that CI should automatically detect problems instead of allowing broken code to continue through the pipeline.

---

## 7. GitHub Actions Cache

I also practiced using:

```yaml
uses: actions/cache@v4
```

I configured a cache for APT packages.

Example:

```yaml
path: /var/cache/apt/archives
```

The cache is stored outside the repository and can be restored on later workflow runs when the cache key matches.

Caching can help reduce repeated dependency downloads and improve workflow execution time.

---

## Workflows Created

```text
.github/workflows/
├── secrets.yml
├── artifacts.yml
├── artifact-between-jobs.yml
├── tests.yml
└── cache.yml
```

---

## Day 44 Files

```text
2026/day-44/
├── README.md
├── day-44-secrets-artifacts.md
└── test_script.sh
```

---

## Key Learnings

Today I understood that a CI pipeline is not just about running a few commands.

A useful CI pipeline should also:

- Keep secrets secure
- Run automated tests
- Fail when tests fail
- Store useful artifacts
- Share outputs between jobs
- Use caching when appropriate

This made my GitHub Actions workflow feel much closer to a real CI pipeline.

---

## Technologies Used

- Ubuntu
- Git
- GitHub
- GitHub Actions
- YAML
- Bash
- GitHub Secrets
- GitHub Artifacts
- GitHub Actions Cache

---

## Conclusion

Today I moved from basic GitHub Actions workflows toward a more practical CI pipeline.

I tested both successful and failed builds, worked with secrets and artifacts, and learned how different jobs can communicate using artifacts.

The next step is to use these concepts with Docker and securely authenticate with Docker Hub.