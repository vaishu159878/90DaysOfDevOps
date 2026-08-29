# Day 61 – Introduction to Terraform 🚀

Today I started learning **Terraform** and Infrastructure as Code.

Until now, I was mostly working with Docker, CI/CD and Kubernetes. Today I went one step deeper and learned how infrastructure itself can be created and managed using code.

For this task, I used Terraform with AWS and created an **S3 bucket** and an **EC2 instance**.

---

## What I Learned Today

### 1. Infrastructure as Code

Infrastructure as Code (IaC) means creating and managing infrastructure using code instead of doing everything manually from the AWS Console.

With Terraform, I can describe the infrastructure I want in a `.tf` file and Terraform takes care of creating or changing the resources.

This makes infrastructure easier to repeat, manage and track.

---

## 2. Terraform Setup

First, I installed Terraform and checked that it was working:

```bash
terraform -version
````

Then I configured AWS CLI:

```bash
aws configure
```

I verified my AWS access using:

```bash
aws sts get-caller-identity
```

This confirmed that Terraform and AWS CLI were connected to my AWS account.

---

## 3. My First Terraform Project

I created a project directory:

```bash
mkdir terraform-basics
cd terraform-basics
```

Then I created:

```text
main.tf
```

My Terraform configuration contained the AWS provider and an S3 bucket.

Example:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}

resource "aws_s3_bucket" "terraform_bucket" {
  bucket = "terraweek-vaishnavi-2026"
}
```

---

## 4. Terraform Init

I started the Terraform project using:

```bash
terraform init
```

Terraform downloaded the AWS provider required for the project.

It also created:

```text
.terraform/
.terraform.lock.hcl
```

The `.terraform/` directory contains Terraform's local working files and downloaded provider plugins.

The `.terraform.lock.hcl` file helps Terraform keep track of the provider version and checksums.

---

## 5. Terraform Plan

Before creating anything, I used:

```bash
terraform plan
```

This allowed me to see what Terraform was going to create before actually making the changes.

For the S3 bucket, Terraform showed:

```text
Plan: 1 to add, 0 to change, 0 to destroy.
```

This was useful because I could review the changes before applying them.

---

## 6. Terraform Apply

I created the S3 bucket using:

```bash
terraform apply
```

After confirming with `yes`, Terraform created the bucket in AWS.

Then I added an EC2 instance to the same Terraform configuration.

My EC2 resource looked like:

```hcl
resource "aws_instance" "terraform_ec2" {
  ami           = "YOUR_AMI_ID"
  instance_type = "t3.micro"

  tags = {
    Name = "TerraWeek-Day1"
  }
}
```

I initially tried `t2.micro`, but AWS returned an error saying that the instance type was not eligible for Free Tier in my account.

I checked the available Free Tier eligible instance types and used `t3.micro`.

---

## 7. AWS Resources

Using Terraform, I created:

* S3 bucket
* EC2 instance

I also checked the AWS Console to verify that the resources were created successfully.

The EC2 instance was running with the tag:

```text
TerraWeek-Day1
```

---

## 8. Terraform State

One of the important things I learned today was the Terraform state file.

After applying the configuration, Terraform created:

```text
terraform.tfstate
```

I used:

```bash
terraform show
```

to see the current Terraform state in a readable format.

I also used:

```bash
terraform state list
```

It showed the resources Terraform was managing:

```text
aws_instance.terraform_ec2
aws_s3_bucket.terraform_bucket
```

I then checked individual resources using:

```bash
terraform state show aws_s3_bucket.terraform_bucket
```

and:

```bash
terraform state show aws_instance.terraform_ec2
```

The state file helps Terraform understand which resources it is already managing.

Because Terraform had the S3 bucket recorded in its state, when I added the EC2 instance and ran:

```bash
terraform plan
```

Terraform knew that the S3 bucket already existed and only planned to create the EC2 instance.

---

## 9. What Does the State File Contain?

The Terraform state file contains information about the infrastructure Terraform manages.

It can contain things such as:

* Resource IDs
* Resource types
* Resource attributes
* Provider information
* Relationships between resources

Terraform uses this information to compare the desired configuration with the current infrastructure.

---

## 10. Why Should We Not Edit the State File Manually?

I learned that the state file should be managed by Terraform itself.

Manually changing the state can make Terraform's understanding of the infrastructure incorrect or inconsistent with the actual AWS resources.

For real projects, Terraform state should also be protected because it can contain sensitive infrastructure information.

I added the state files to `.gitignore` so they are not committed to Git.

---

## 11. Changing the Infrastructure

After creating the EC2 instance, I changed the Name tag from:

```text
TerraWeek-Day1
```

to:

```text
TerraWeek-Modified
```

Then I ran:

```bash
terraform plan
```

Terraform showed:

```text
~ update in-place
```

and:

```text
Plan: 0 to add, 1 to change, 0 to destroy.
```

The `~` symbol means Terraform will modify an existing resource.

I then applied the change:

```bash
terraform apply
```

Terraform successfully updated the EC2 instance.

I checked the AWS Console again and saw:

```text
TerraWeek-Modified
```

---

## 12. Terraform Plan Symbols

| Symbol | Meaning                     |
| ------ | --------------------------- |
| `+`    | Create a new resource       |
| `~`    | Modify an existing resource |
| `-`    | Destroy a resource          |

This helped me understand how Terraform shows the changes it is going to make before applying them.
                      |

---

## 13. Destroying the Infrastructure

After completing the testing, I cleaned up the resources using:

```bash
terraform destroy
```

Terraform showed me the resources it was going to remove and asked for confirmation.

After entering `yes`, Terraform removed the resources managed by this Terraform configuration.

This helped me understand the complete Terraform workflow:

```text
Write configuration
        ↓
terraform init
        ↓
terraform plan
        ↓
terraform apply
        ↓
Modify configuration
        ↓
terraform plan
        ↓
terraform apply
        ↓
terraform destroy
```

My `.gitignore` contains:

```gitignore
.terraform/
*.tfstate
*.tfstate.backup
```

---

## 14. What I Learned About Terraform vs Other Tools

### Terraform vs CloudFormation

Terraform can manage infrastructure across different cloud providers, while CloudFormation is AWS's infrastructure-as-code service.

### Terraform vs Ansible

Terraform is mainly used to provision and manage infrastructure.

Ansible is commonly used for configuration management and automating tasks on existing machines.

### Terraform vs Pulumi

Both Terraform and Pulumi can manage cloud infrastructure.

Terraform uses HCL for its configuration, while Pulumi allows infrastructure to be defined using programming languages such as Python, TypeScript, Go and C#.

---

## 15. Declarative and Cloud-Agnostic

Terraform is **declarative** because I describe the infrastructure I want instead of writing every individual step needed to create it.

For example, I can say that I want an EC2 instance with a specific AMI and instance type, and Terraform determines what actions are needed.

Terraform is also **cloud-agnostic** because it can work with multiple cloud providers through providers.

---

