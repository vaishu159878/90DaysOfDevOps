# GitHub Actions Runners

## Overview

In this task, I explored how GitHub Actions workflows actually run using **runners**.

I worked with both:

- GitHub-hosted runners
- Self-hosted runners
- Runner labels
- AWS EC2
- GitHub Actions workflows

The main goal was to understand the difference between a runner managed by GitHub and a runner running on my own EC2 instance.

---

## What is a Runner?

A runner is the machine that executes the steps defined inside a GitHub Actions workflow.

For example:

```text
GitHub Repository
       ↓
GitHub Actions
       ↓
Runner
       ↓
Workflow Job
```

---

## GitHub-Hosted Runners

I created a workflow with three jobs using different operating systems:

```yaml
runs-on: ubuntu-latest
```

```yaml
runs-on: windows-latest
```

```yaml
runs-on: macos-latest
```

Each job printed:

- Operating system
- Hostname
- Current user

The three jobs were able to run in parallel.

GitHub manages these runners, so I don't need to create or maintain the machines myself.

---

## Pre-installed Tools

I also checked some tools available on the Ubuntu GitHub-hosted runner.

```bash
docker --version
python3 --version
node --version
git --version
```

GitHub-hosted runners come with many commonly used tools already installed.

This is useful because I don't have to spend time installing every basic tool before running a workflow.

---

## Self-Hosted Runner

For the self-hosted part, I created an **Ubuntu EC2 instance on AWS**.

I registered the EC2 instance as a self-hosted GitHub Actions runner.

The flow was:

```text
GitHub Repository
       ↓
GitHub Actions
       ↓
AWS EC2
       ↓
Self-Hosted Runner
       ↓
Workflow Job
```

After starting the runner, GitHub showed my runner as:

```text
Idle
```

with a green status.

---

## Running a Job on My EC2

I created a workflow using:

```yaml
runs-on: self-hosted
```

The workflow printed:

```bash
hostname
whoami
pwd
```

It also created a file on the EC2 machine and then verified that the file existed.

For example:

```bash
echo "Hello from my self-hosted runner!" > day42-runner-test.txt
```

Then I checked:

```bash
ls -l day42-runner-test.txt
cat day42-runner-test.txt
```

The file was present on my EC2 instance after the workflow completed.

This helped me understand that the GitHub Actions job was actually executing on my own EC2 machine.

---

## Runner Labels

I added a custom label:

```text
my-linux-runner
```

Then I changed the workflow to:

```yaml
runs-on: [self-hosted, my-linux-runner]
```

The workflow successfully picked up the job using the label.

Labels are useful when there are multiple self-hosted runners because they allow workflows to target runners with specific characteristics or installed tools.

---

## GitHub-Hosted vs Self-Hosted

| | GitHub-Hosted | Self-Hosted |
|---|---|---|
| Managed by | GitHub | Me / Organization |
| Infrastructure | Provided by GitHub | My own infrastructure |
| Setup | Very easy | Requires setup and maintenance |
| Tools | Many tools are pre-installed | I install the tools I need |
| Customization | More limited | More control |
| Good for | Standard CI/CD workflows | Custom environments and infrastructure |
| Security | GitHub manages runner infrastructure | I am responsible for securing the runner |

---

## One Problem I Faced

While pushing my changes from EC2, Git asked me for a GitHub username and password.

I checked my Git remote and found that it was using HTTPS.

I changed it to SSH:

```bash
git remote set-url origin git@github.com:vaishu159878/90DaysOfDevOps.git
```

Then I tested:

```bash
ssh -T git@github.com
```

After SSH authentication was configured, I was able to push my changes successfully.

This was a small issue, but it helped me understand the importance of configuring Git authentication correctly on a server.

---

## Files

The workflows created for this task are:

```text
.github/
└── workflows/
    ├── runners.yml
    ├── tools.yml
    └── self-hosted.yml
```

Documentation:

```text
day-42-runners.md
```

---

## What I Understood

The main thing I understood from this task is that a GitHub Actions workflow needs a **runner** to execute its jobs.

With GitHub-hosted runners:

```text
GitHub
  ↓
GitHub-managed runner
  ↓
Job
```

With a self-hosted runner:

```text
GitHub
  ↓
My AWS EC2
  ↓
Self-hosted runner
  ↓
Job
```

Running a workflow directly on my own EC2 instance made the concept much clearer than just reading about runners.

---

