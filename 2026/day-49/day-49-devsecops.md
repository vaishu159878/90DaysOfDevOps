# Day 49 – DevSecOps: Securing the CI/CD Pipeline

## Overview

This project adds security checks to the GitHub Actions CI/CD pipeline so that vulnerabilities can be detected before changes reach deployment.

The security workflow covers:

- Docker image vulnerability scanning with Trivy
- Dependency vulnerability checking for pull requests
- GitHub Secret Scanning
- Push Protection
- Least-privilege workflow permissions
- Security checks before Docker push and deployment

## Architecture

```text
Developer
    |
    | Pull Request
    v
GitHub PR
    |
    +--> Build & Test
    |
    +--> Dependency Review
    |
    +--> PR Checks
    |
  Merge
    |
    v
main
    |
    +--> Build & Test
    |
    +--> Docker Build
    |
    +--> Trivy Scan
    |       |
    |       +--> HIGH/CRITICAL found --> FAIL
    |       |
    |       +--> No blocking findings
    |
    +--> Docker Push
    |
    +--> Deploy
```

## 1. Trivy Docker Image Scanning

Trivy was added to the reusable Docker workflow.

The scan is configured to fail the workflow when HIGH or CRITICAL vulnerabilities are detected.

```yaml
- name: Scan Docker Image for Vulnerabilities
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: '${{ inputs.image_name }}:${{ inputs.tag }}'
    format: 'table'
    exit-code: '1'
    severity: 'CRITICAL,HIGH'
```

This creates a security gate before the Docker image can be pushed and deployed.

## 2. Vulnerability Found During Testing

The first Docker image used a Debian-based Python slim image.

Trivy reported:

```text
Total: 44
HIGH: 44
CRITICAL: 0
```

The findings were in Debian operating-system packages, including `util-linux`.

The pipeline failed as expected because the configured security gate treats HIGH and CRITICAL findings as blocking issues.

This demonstrated that the Trivy check was actually enforcing the configured security policy.

## 3. Changing the Container Base Image

After investigating the scan results, the Dockerfile was changed to an Alpine-based Python image:

```dockerfile
FROM python:3.12.14-alpine3.24

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

The image was rebuilt locally and scanned again.

## 4. Local Trivy Verification

The final local scan used:

```bash
trivy image --severity HIGH,CRITICAL github-actions-capstone:security-test
```

Trivy detected:

```text
Alpine 3.24.2
```

The report showed:

```text
Vulnerabilities: 0
```

The Python package entries also showed:

```text
0
```

Therefore:

```text
HIGH: 0
CRITICAL: 0
```

The image passed the configured security gate.

## 5. Dependency Review for Pull Requests

The PR workflow includes GitHub's Dependency Review Action.

```yaml
- name: Dependency Review
  uses: actions/dependency-review-action@v4
  with:
    fail-on-severity: critical
```

The repository Dependency Graph was enabled so that dependency review could analyze the project's dependencies.

## 6. Secret Scanning and Push Protection

GitHub Secret Scanning and Push Protection were enabled in the repository's security settings.

These controls help detect secrets that could accidentally be committed or pushed to the repository.

## 7. Least-Privilege Workflow Permissions

The workflows were updated with:

```yaml
permissions:
  contents: read
```

This limits the default permissions available to the workflow and follows the principle of least privilege.

## 8. Pull Request Security Flow

```text
Pull Request
     |
     v
Build & Test
     |
     v
Dependency Review
     |
     v
PR Checks
```

The pull-request stage focuses on validation and security checks and does not push the Docker image to Docker Hub.

## 9. Main Branch Security Flow

```text
main
 |
 v
Build & Test
 |
 v
Docker Build
 |
 v
Trivy Vulnerability Scan
 |
 +---- HIGH/CRITICAL found ----> FAIL
 |
 +---- No blocking findings ---> Docker Push
                                      |
                                      v
                                    Deploy
```

The security scan occurs before the image is pushed and deployed.

## 10. Final GitHub Actions Result

After changing the Docker base image and merging the fix, the Main Pipeline completed successfully.

Successful jobs:

```text
build-test     ✅
docker-latest  ✅
prepare        ✅
docker-sha     ✅
deploy         ✅
```

The complete flow therefore became:

```text
Build
  ↓
Test
  ↓
Docker Build
  ↓
Trivy Security Scan
  ↓
Docker Push
  ↓
Deploy
```

## 11. What I Learned

### DevSecOps

Security can be integrated into the development and deployment workflow instead of being treated as a separate final step.

### Trivy

Trivy can scan container images for known vulnerabilities and can be configured to fail CI when selected severity levels are detected.

### Dependency Review

Pull requests can be checked for dependency vulnerabilities before changes are merged.

### Secret Protection

Secret Scanning and Push Protection help prevent credentials and sensitive information from being committed to the repository.

### Least Privilege

GitHub Actions workflows should use the minimum permissions required.

### Troubleshooting

A security pipeline can fail because of vulnerabilities in the base image even when application Python packages have no reported vulnerabilities. The actual findings should be investigated and the replacement image should be verified with Trivy before updating the CI/CD pipeline.


## Conclusion

This project turned a normal CI/CD pipeline into a more security-aware DevSecOps pipeline.

The key addition was making the vulnerability scan a blocking quality gate:

```text
Code
 ↓
Test
 ↓
Security Check
 ↓
Build/Push
 ↓
Deploy
```

The pipeline now checks dependencies and container vulnerabilities before deployment, while GitHub Secret Scanning and Push Protection provide additional protection for sensitive information.

## Repository

GitHub:
https://github.com/vaishu159878/github-actions-capstone

Docker Hub:
https://hub.docker.com/r/vaishnavinalawade/github-actions-capstone
