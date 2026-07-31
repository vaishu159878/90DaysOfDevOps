# Day 38 – YAML Basics

## Objective

Learn the fundamentals of YAML syntax by creating YAML files, understanding indentation, lists, nested objects, multiline strings, and validating YAML files.

---

# Task 1 – Key Value Pairs

Created `person.yaml` with:

```yaml
---
name: Vaishnavi Nalawade
role: Fresher
experience_years: 0
learning: true
```

Verified using:

```bash
cat person.yaml
```

---

# Task 2 – Lists

Added tools and hobbies.

```yaml
tools:
  - Linux
  - Git
  - Docker
  - AWS
  - GitHub Actions

hobbies: [Reading, Listening to Music]
```

### Two ways to write lists

### Block Style

```yaml
tools:
  - Linux
  - Git
```

### Inline Style

```yaml
tools: Reading, Listening to Music]
```

---

# Task 3 – Nested Objects

Created `server.yaml`

```yaml
---
server:
  name: web-server
  ip: 192.168.1.10
  port: 80

database:
  host: localhost
  name: app_db
  credentials:
    user: admin
    password: password123
```

### What happens if tabs are used?

YAML does not allow tabs for indentation.

Example error:

```
found character '\t' that cannot start any token
```

---

# Task 4 – Multi-line Strings

### Pipe (|)

```yaml
startup_script_pipe: |
  #!/bin/bash
  echo "Starting application..."
  systemctl start nginx
```

Preserves line breaks exactly.

### Fold (>)

```yaml
startup_script_fold: >
  #!/bin/bash
  echo "Starting application..."
  systemctl start nginx
```

Combines lines into one paragraph while preserving blank lines.

### When to use?

- `|` → Shell scripts, certificates, configuration files.
- `>` → Long descriptions, comments, documentation.

---

# Task 5 – YAML Validation

Installed yamllint:

```bash
sudo apt update
sudo apt install yamllint -y
```

Validate files:

```bash
yamllint person.yaml
yamllint server.yaml
```

Intentional indentation error:

```yaml
tools:
- Docker
  - Git
```

Example error:

```
syntax error: expected <block end>, but found '-'
```

Fixed indentation and validated successfully.

---

# Task 6 – Spot the Difference

Correct:

```yaml
tools:
  - docker
  - git
```

Broken:

```yaml
tools:
- docker
  - git
```

### Issue

The list items are inconsistently indented.

YAML requires consistent indentation.

Correct version:

```yaml
tools:
  - docker
  - git
```

---

# What I Learned

1. YAML is indentation-sensitive and uses spaces only.
2. YAML supports block and inline lists, nested objects, and multiline strings.
3. Always validate YAML using tools like `yamllint` before using it in CI/CD pipelines.

---

# Commands Used

```bash
cat person.yaml
cat server.yaml

sudo apt update
sudo apt install yamllint -y

yamllint person.yaml
yamllint server.yaml
```