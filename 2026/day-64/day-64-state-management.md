# Day 64 – Terraform State Management and Remote Backends

## Overview

Today I learned about **Terraform State Management** and why state is important when working with Infrastructure as Code.

---

---

# Task 1 – Inspect Terraform State

I created a small AWS infrastructure using Terraform.

Resources created:

- VPC
- Public Subnet
- Internet Gateway
- Route Table
- Route Table Association
- Security Group
- EC2 Instance

I checked the resources tracked by Terraform using:

```bash
terraform state list
```

Terraform was tracking **7 resources**.

I also used:

```bash
terraform show
```

and:

```bash
terraform state show aws_instance.main
```

The EC2 state contains many more attributes than I manually defined, such as:

- Instance ID
- AMI
- Instance type
- Private IP
- Public IP
- Availability Zone
- Security groups
- Network interface
- Root block device
- Metadata options
- Tags
- DNS information

I also checked the state serial number:

```bash
grep '"serial"' terraform.tfstate
```

My state showed:

```text
"serial": 8
```

The serial number represents the revision/version of the Terraform state snapshot.

---

# Task 2 – Configure S3 Remote Backend

Keeping Terraform state only on a local machine is risky.

I created an S3 bucket for remote state and enabled versioning.

### S3 Backend

```hcl
terraform {
  backend "s3" {
    bucket         = "terraweek-state-vaishnavi"
    key            = "dev/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "terraweek-state-lock"
    encrypt        = true
  }
}
```

I then migrated the existing local state:

```bash
terraform init -migrate-state
```

Terraform asked whether I wanted to copy the existing state to the new backend, and I selected:

```text
yes
```

Terraform successfully configured the S3 backend.

I verified the state in S3:

```bash
aws s3 ls s3://terraweek-state-vaishnavi/dev/
```

Result:

```text
terraform.tfstate
```

I also verified that S3 versioning was enabled:

```bash
aws s3api get-bucket-versioning \
  --bucket terraweek-state-vaishnavi
```

Result:

```json
{
    "Status": "Enabled"
}
```

Finally:

```bash
terraform plan
```

returned:

```text
No changes. Your infrastructure matches the configuration.
```

This confirmed that the state migration was successful.

> Note: My Terraform version reports `dynamodb_table` as deprecated and recommends `use_lockfile`. I kept the DynamoDB configuration because this exercise specifically focuses on demonstrating DynamoDB-based state locking.

---

# Task 3 – Test Terraform State Locking

State locking prevents multiple Terraform operations from modifying the same state at the same time.

I used two terminals and ran Terraform operations against the same remote state.

The second Terraform operation produced:

```text
Error: Error acquiring the state lock
```

The error included:

```text
ConditionalCheckFailedException
```

and showed the lock information:

```text
Lock ID: 62b0d4ae-40d9-6213-57e5-49686c036416
Path: terraweek-state-vaishnavi/dev/terraform.tfstate
Operation: OperationTypeApply
```

This demonstrated that DynamoDB was preventing the second Terraform operation from acquiring the existing state lock.

### Why locking is important

Without locking, two engineers could run Terraform at the same time and update the same state.

This could result in:

- Conflicting changes
- State corruption
- Unexpected infrastructure changes
- Difficult recovery

State locking is therefore very important in team environments.

---

# Task 4 – Import an Existing AWS Resource

I created an S3 bucket outside Terraform:

```text
terraweek-import-test-vaishnavi
```

Then I added the following Terraform configuration:

```hcl
resource "aws_s3_bucket" "imported" {
  bucket = "terraweek-import-test-vaishnavi"
}
```

Before importing, Terraform planned to create the bucket because it existed in AWS but was not present in Terraform state.

I imported it using:

```bash
terraform import aws_s3_bucket.imported terraweek-import-test-vaishnavi
```

Terraform returned:

```text
Import successful!
```

I verified the imported resource:

```bash
terraform state list
```

The bucket appeared in Terraform state.

I then renamed the Terraform resource to `logs_bucket` as part of the state surgery task.

After the import and configuration matched, I ran:

```bash
terraform plan
```

Result:

```text
No changes. Your infrastructure matches the configuration.
```

### `terraform import` vs creating a resource

`terraform apply` creates a new resource based on Terraform configuration.

`terraform import` does not create a new resource. It brings an already-existing resource into Terraform state so that Terraform can manage it.

---

# Task 5 – Terraform State Surgery

## `terraform state mv`

Initially the resource address was:

```text
aws_s3_bucket.imported
```

I moved it to:

```text
aws_s3_bucket.logs_bucket
```

using:

```bash
terraform state mv aws_s3_bucket.imported aws_s3_bucket.logs_bucket
```

Terraform successfully moved the resource.

I also updated `main.tf` to use:

```hcl
resource "aws_s3_bucket" "logs_bucket" {
  bucket = "terraweek-import-test-vaishnavi"
}
```

Then:

```bash
terraform plan
```

returned:

```text
No changes.
```

### When would I use `state mv`?

I would use `terraform state mv` when changing the Terraform resource address but keeping the same real infrastructure.

For example:

- Renaming a resource
- Reorganizing Terraform code
- Moving resources into modules
- Changing module/resource structure

---

## `terraform state rm`

I removed the bucket from Terraform state:

```bash
terraform state rm aws_s3_bucket.logs_bucket
```

Terraform removed it from state successfully.

I verified that the bucket still existed in AWS:

```bash
aws s3 ls | grep terraweek-import-test-vaishnavi
```

The bucket was still there.

This demonstrated that:

> `terraform state rm` removes a resource from Terraform state but does not destroy the actual AWS resource.

I then imported the bucket again:

```bash
terraform import aws_s3_bucket.logs_bucket terraweek-import-test-vaishnavi
```

The import was successful.

Finally:

```bash
terraform plan
```

returned:

```text
No changes. Your infrastructure matches the configuration.
```

---

# Task 6 – Simulate and Fix State Drift

State drift happens when the real infrastructure is changed outside Terraform.

My Terraform configuration defined the EC2 Name tag as:

```text
day-64-state-instance
```

I manually changed the EC2 Name tag outside Terraform to:

```text
ManuallyChanged
```

I verified the manual change using AWS CLI.

Then I ran:

```bash
terraform plan
```

Terraform detected the drift:

```text
~ "Name" = "ManuallyChanged" -> "day-64-state-instance"
```

The plan showed:

```text
Plan: 0 to add, 1 to change, 0 to destroy.
```

This showed that Terraform detected the difference between the desired configuration and the actual AWS infrastructure.

I chose to restore the infrastructure to the Terraform configuration:

```bash
terraform apply
```

After applying the change, the EC2 Name tag was restored to:

```text
day-64-state-instance
```

I then ran:

```bash
terraform plan
```

Result:

```text
No changes. Your infrastructure matches the configuration.
```

### How teams can prevent drift

Teams can reduce infrastructure drift by:

- Limiting manual changes through the AWS Console
- Using Terraform as the source of truth
- Managing infrastructure through CI/CD pipelines
- Restricting production console access
- Reviewing Terraform changes through pull requests
- Running regular Terraform plans
- Using remote state and state locking

---


