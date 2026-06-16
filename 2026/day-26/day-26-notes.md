# Day 26 – GitHub CLI: Manage GitHub from Your Terminal

## Overview

Today I learned how to use GitHub CLI (`gh`) to manage GitHub directly from the terminal. Before this, I mostly used GitHub through the browser, but GitHub CLI showed me that many tasks can be completed faster without leaving the command line.

I completed this task on an AWS EC2 Ubuntu instance, which also gave me more practice working in a Linux environment.

---

## Task 1: Install and Authenticate

### What I Did

* Installed GitHub CLI on Ubuntu
* Logged into my GitHub account using device authentication
* Verified my account using `gh auth status`

### Authentication Methods Supported by gh

GitHub CLI supports:

* Browser-based authentication
* Personal Access Tokens (PAT)
* SSH authentication
* GitHub Enterprise authentication

### Observation

The login process was simple, and I liked that I could authenticate from a remote AWS server using the device login method.

---

## Task 2: Working with Repositories

### What I Did

* Created a public repository directly from the terminal
* Cloned the repository using `gh repo clone`
* Viewed repository details using GitHub CLI
* Listed repositories from the terminal
* Opened the repository in the browser using a command

### Observation

Repository management felt much faster compared to switching between the browser and terminal.

---

## Task 3: Issues

### What I Did

* Created an issue from the terminal
* Listed open issues
* Viewed issue details
* Closed the issue

### How could gh issue be used in automation?

I can use `gh issue` in scripts to:

* Automatically create issues when a server alert occurs
* Open issues when CI/CD pipelines fail
* Create maintenance or monitoring tasks automatically
* Track infrastructure problems from automation scripts

### Observation

This feature can save time and help teams track problems automatically.

---

## Task 4: Pull Requests

### What I Did

* Created a feature branch
* Added changes and committed them
* Pushed the branch to GitHub
* Created a Pull Request using GitHub CLI
* Viewed PR details
* Merged the Pull Request from the terminal

### What merge methods does gh pr merge support?

GitHub CLI supports:

* Merge Commit (`--merge`)
* Squash Merge (`--squash`)
* Rebase Merge (`--rebase`)

### How would you review someone else's PR using gh?

I can:

* Checkout the PR locally using `gh pr checkout`
* View PR details using `gh pr view`
* Add comments using `gh pr review`
* Approve the PR
* Request changes

### Observation

Creating and merging a Pull Request without opening GitHub was my favorite part of today's task.

---

## Task 5: GitHub Actions & Workflows

### What I Did

* Listed workflow runs on a public repository
* Viewed the details of a workflow run
* Explored workflow execution information

### How could gh run and gh workflow be useful in CI/CD?

These commands can help:

* Monitor pipeline status
* Check build and deployment results
* View workflow logs
* Re-run failed workflows
* Automate deployment monitoring

### Observation

I haven't learned GitHub Actions yet, but this gave me a good preview of how CI/CD pipelines can be monitored from the terminal.

---

## Task 6: Useful gh Commands

Commands I explored:

```bash
gh api user
gh gist create
gh release create
gh alias set
gh search repos devops
```

### Observation

GitHub CLI is more powerful than I expected. It can interact with many GitHub features directly from the terminal.

