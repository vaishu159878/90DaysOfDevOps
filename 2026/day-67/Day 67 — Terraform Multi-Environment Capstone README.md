# 🚀 Day 67 — TerraWeek Terraform Capstone

## 📌 Overview

For Day 67 of my #90DaysOfDevOps journey, I completed a Terraform capstone project where I brought together the concepts I learned throughout TerraWeek.

The goal was to build **multiple AWS environments using one Terraform codebase**, reusable modules, and Terraform workspaces.

I created:

- Development environment
- Staging environment
- Production environment

The main idea was:

> **One codebase → reusable modules → multiple environments**

---

## 🎯 What I Built

Using Terraform, I created three reusable custom modules:

### 1. VPC Module

The VPC module creates:

- VPC
- Public subnet
- Internet Gateway
- Route table
- Route table association

### 2. Security Group Module

The Security Group module creates:

- Security Group
- Dynamic ingress rules
- Allow-all egress

The ingress ports are controlled through variables for each environment.

### 3. EC2 Instance Module

The EC2 module creates:

- EC2 instance
- Environment-specific instance configuration
- Environment-based resource tags

---

## 🏗️ Project Structure

```text
terraweek-capstone/
│
├── main.tf
├── variables.tf
├── outputs.tf
├── providers.tf
├── locals.tf
│
├── dev.tfvars
├── staging.tfvars
├── prod.tfvars
│
├── .gitignore
├── day-67-terraweek-capstone.md
│
└── modules/
    │
    ├── vpc/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    │
    ├── security-group/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    │
    └── ec2-instance/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

---

## 🌍 Terraform Workspaces

I used Terraform workspaces to manage the three environments:

```text
dev
staging
prod
```

I used:

```hcl
terraform.workspace
```

to identify the current environment.

For example:

```hcl
locals {
  environment = terraform.workspace
}
```

This allowed the same Terraform configuration to be reused for all three environments.

---

## 🔧 Environment Configuration

Each environment has its own `.tfvars` file.

| Environment | VPC CIDR | Subnet CIDR | SSH | HTTP | HTTPS |
|---|---|---|---|---|---|
| Dev | 10.0.0.0/16 | 10.0.1.0/24 | ✅ | ✅ | ❌ |
| Staging | 10.1.0.0/16 | 10.1.1.0/24 | ✅ | ✅ | ✅ |
| Prod | 10.2.0.0/16 | 10.2.1.0/24 | ❌ | ❌ | ✅ |

Using different CIDR ranges helped keep the environments separated and prevented overlapping networks.

---

## 🧩 How Terraform Connects Everything

The modules are connected through their outputs and inputs.

```text
                Terraform Root Module
                        │
                        ▼
                  VPC Module
                        │
                 vpc_id / subnet_id
                        │
                        ▼
              Security Group Module
                        │
                      sg_id
                        │
                        ▼
                EC2 Instance Module
```

Terraform automatically understands the dependencies through these module references.

---

## 🚀 Deployment Process

### Initialize Terraform

```bash
terraform init
```

### Validate configuration

```bash
terraform validate
```

### Create workspaces

```bash
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod
```

### Deploy DEV

```bash
terraform workspace select dev

terraform plan -var-file="dev.tfvars"

terraform apply -var-file="dev.tfvars"
```

### Deploy STAGING

```bash
terraform workspace select staging

terraform plan -var-file="staging.tfvars"

terraform apply -var-file="staging.tfvars"
```

### Deploy PROD

```bash
terraform workspace select prod

terraform plan -var-file="prod.tfvars"

terraform apply -var-file="prod.tfvars"
```

---

## 🔍 Verification

I verified each environment using:

```bash
terraform output
```

and:

```bash
terraform state list
```

Each workspace had its own:

- VPC
- Subnet
- Internet Gateway
- Route Table
- Security Group
- EC2 Instance

I also verified the resources in AWS.

---

## 📊 Deployment Results

### DEV

```text
VPC:     vpc-044ca2468c73d5fbe
Subnet:  subnet-098a99e80da5d7d52
EC2:     i-0036ff21f8ad33450
```

### STAGING

```text
VPC:     vpc-0f66cc378900f8b9b
Subnet:  subnet-05ba9a1ecb6409c0d
EC2:     i-084f34221a7d1de57
```

### PROD

```text
VPC:     vpc-08ec5aa9f5cc1906c
Subnet:  subnet-0e9a4cc1eec3f5fbb
EC2:     i-018a324ab550845c9
```

---

## 🧹 Cleanup

After verifying all three environments, I destroyed the infrastructure to avoid leaving unnecessary AWS resources running.

I destroyed the environments in reverse order:

```bash
terraform workspace select prod
terraform destroy -var-file="prod.tfvars"

terraform workspace select staging
terraform destroy -var-file="staging.tfvars"

terraform workspace select dev
terraform destroy -var-file="dev.tfvars"
```

After that, I switched back to the default workspace:

```bash
terraform workspace select default
```

Then deleted the environment workspaces:

```bash
terraform workspace delete dev
terraform workspace delete staging
terraform workspace delete prod
```

Finally:

```bash
terraform workspace list
```

Only the `default` workspace remained.

---

## 💡 Free Tier Consideration

During the deployment, AWS rejected the original EC2 instance type because it was not eligible for my account's current Free Tier configuration.

Instead of continuing with an instance type that could cause unnecessary charges, I used a Free Tier-eligible instance type for the environments.

The main objective of the exercise — Terraform modules, workspaces, multi-environment configuration, deployment, verification, and cleanup — remained the same.

---

## 📚 Best Practices I Practiced

### 1. Reusable Modules

Instead of putting everything inside one `main.tf`, I separated infrastructure into focused modules.

### 2. Variables

Environment-specific values are passed through `.tfvars` files instead of hardcoding them.

### 3. Workspaces

Terraform workspaces were used to maintain separate states for different environments.

### 4. Consistent Naming

Resources follow a naming pattern:

```text
<project>-<environment>-<resource>
```

Example:

```text
terraweek-dev-server
terraweek-staging-server
terraweek-prod-server
```

### 5. Tagging

Resources are tagged with information such as:

```text
Project
Environment
ManagedBy
Workspace
```

### 6. Validation Before Deployment

I followed:

```text

terraform validate
       ↓
terraform plan
       ↓
terraform apply
```

### 7. Cleanup

After verification, I destroyed the infrastructure so that unused AWS resources would not continue running.

---

## 🧠 What I Learned

This project helped me understand how Terraform can be used beyond creating individual AWS resources.

The biggest thing I learned was how **modules and workspaces can be combined to manage multiple environments using the same Terraform codebase**.

I also got more comfortable with:

- Terraform state
- Terraform workspaces
- Custom modules
- Module inputs and outputs
- Dynamic blocks
- Variables
- `.tfvars`
- Data sources
- Resource dependencies
- AWS networking
- EC2 provisioning
- Terraform lifecycle
- Infrastructure cleanup

---


---


**One codebase. Three environments. Reusable infrastructure.**

Still learning, still building, and moving one step closer to becoming a better DevOps engineer. 🚀

---

## 🛠️ Technologies Used

- Terraform
- AWS
- EC2
- VPC
- Security Groups
- Terraform Modules
- Terraform Workspaces
- HCL
- Linux
- Git & GitHub

---

