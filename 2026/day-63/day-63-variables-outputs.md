# Day 63 - Variables, Outputs, Data Sources and Expressions
---

## 1. Terraform Variables

I created a `variables.tf` file and added variables for the values that can change between environments.

### variables.tf

```hcl
variable "region" {
  type    = string
  default = "ap-south-1"
}

variable "vpc_cidr" {
  type    = string
  default = "10.0.0.0/16"
}

variable "subnet_cidr" {
  type    = string
  default = "10.0.1.0/24"
}

variable "instance_type" {
  type    = string
  default = "t3.micro"
}

variable "project_name" {
  type = string
}

variable "environment" {
  type    = string
  default = "dev"
}

variable "allowed_ports" {
  type    = list(number)
  default = [22, 80, 443]
}

variable "extra_tags" {
  type    = map(string)
  default = {}
}
```

The `project_name` variable does not have a default value, so Terraform requires me to provide it.

### Variable Types

The five main Terraform variable types I learned are:

| Type | Example | Use |
|---|---|---|
| string | `"dev"` | Text values |
| number | `10` | Numeric values |
| bool | `true` | True/false values |
| list | `["a", "b"]` | Multiple ordered values |
| map | `{Environment = "dev"}` | Key-value pairs |

---

# 2. Variable Files

I created two variable files so that I can use different values for different environments.

## terraform.tfvars

This file is automatically loaded by Terraform.

```hcl
project_name  = "terraweek"
environment   = "dev"
instance_type = "t3.micro"
```

## prod.tfvars

For the production environment:

```hcl
project_name  = "terraweek"
environment   = "prod"
instance_type = "t3.small"

vpc_cidr    = "10.1.0.0/16"
subnet_cidr = "10.1.1.0/24"
```

I tested the production file using:

```bash
terraform plan -var-file="prod.tfvars"
```

This showed me that the same Terraform configuration can be used with different variable values.

---

# 3. Variable Precedence

I learned that Terraform gets variable values from different sources and some sources have higher priority than others.


For example:

```bash
terraform plan
```

uses `terraform.tfvars` automatically.

For production:

```bash
terraform plan -var-file="prod.tfvars"
```

For a command-line override:

```bash
terraform plan -var="instance_type=t2.nano"
```

I also tested an environment variable:

```bash
export TF_VAR_environment="staging"
terraform plan
```

After testing, I removed it:

```bash
unset TF_VAR_environment
```

---

# 4. Data Sources

One of the important changes I made was removing the hardcoded AMI ID.

I used an AWS data source to find an Amazon Linux 2 AMI.

### data.tf

```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  filter {
    name   = "root-device-type"
    values = ["ebs"]
  }
}
```

Then I used the AMI in my EC2 instance:

```hcl
ami = data.aws_ami.amazon_linux.id
```

Terraform found the AMI dynamically.

The AMI returned during my test was:

```text
ami-0beaca59c72740b1b
```

I also created an Availability Zone data source:

```hcl
data "aws_availability_zones" "available" {
  state = "available"
}
```

I used the first available AZ:

```hcl
availability_zone = data.aws_availability_zones.available.names[0]
```

During my test it returned:

```text
ap-south-1a
```

### Resource vs Data Source

A resource is used to create and manage infrastructure.

Example:

```hcl
resource "aws_instance" "web" {
  ...
}
```

A data source is used to read information that already exists in AWS.

Example:

```hcl
data "aws_ami" "amazon_linux" {
  ...
}
```

So, in simple words:

```text
Resource = Create/manage
Data source = Read/fetch
```

---

# 5. Locals

I used locals to avoid repeating common values.

### locals.tf

```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"

  common_tags = merge(var.extra_tags, {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
  })
}
```

This helped me create consistent resource names.

For example:

```text
terraweek-dev-vpc
terraweek-dev-subnet
terraweek-dev-server
terraweek-dev-sg
```

For tags I used:

```hcl
tags = merge(local.common_tags, {
  Name = "${local.name_prefix}-server"
})
```

This makes all my resources have common tags such as:

```text
Project     = terraweek
Environment = dev
ManagedBy   = Terraform
```

---

# 6. Infrastructure Created

Using Terraform, I created the following AWS resources:

```text
1. VPC
2. Internet Gateway
3. Public Subnet
4. Route Table
5. Route Table Association
6. Security Group
7. EC2 Instance
```

Terraform successfully created all 7 resources.

```text
Apply complete! Resources: 7 added, 0 changed, 0 destroyed.
```

---

# 7. Terraform Outputs

I created an `outputs.tf` file to display important information after `terraform apply`.

```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}

output "subnet_id" {
  value = aws_subnet.public.id
}

output "instance_id" {
  value = aws_instance.web.id
}

output "instance_public_ip" {
  value = aws_instance.web.public_ip
}

output "instance_public_dns" {
  value = aws_instance.web.public_dns
}

output "security_group_id" {
  value = aws_security_group.web_sg.id
}
```

I tested:

```bash
terraform output
```

My outputs were:

```text
instance_id = "i-01dc303476f94b940"
instance_public_dns = ""
instance_public_ip = "13.201.47.201"
security_group_id = "sg-06a07e98103ef60a2"
subnet_id = "subnet-0645cab857329fac7"
vpc_id = "vpc-02397a8e7e126a8f2"
```

I also tested a specific output:

```bash
terraform output instance_public_ip
```

Result:

```text
"13.201.47.201"
```

I also tested JSON output:

```bash
terraform output -json
```

This is useful when Terraform output needs to be used by scripts.

---

# 8. Terraform Console

I used `terraform console` to practice Terraform functions and expressions.

Command:

```bash
terraform console
```

## upper()

```hcl
upper("terraweek")
```

Output:

```text
"TERRAWEEK"
```

This converts a string to uppercase.

## join()

```hcl
join("-", ["terra", "week", "2026"])
```

Output:

```text
"terra-week-2026"
```

This joins multiple strings together.

## length()

```hcl
length(["a", "b", "c"])
```

Output:

```text
3
```

This returns the number of elements.

## lookup()

```hcl
lookup({dev = "t2.micro", prod = "t3.small"}, "dev")
```

Output:

```text
"t2.micro"
```

This gets a value from a map.

## cidrsubnet()

```hcl
cidrsubnet("10.0.0.0/16", 8, 1)
```

Output:

```text
"10.0.1.0/24"
```

This calculates a subnet CIDR from a larger CIDR block.

---

# 9. Conditional Expression

I also practiced conditional expressions.

The basic syntax is:

```hcl
condition ? value_if_true : value_if_false
```

Example:

```hcl
var.environment == "prod" ? "t3.small" : "t2.micro"
```

When the environment is `dev`, the result is:

```text
"t2.micro"
```

When the environment is `prod`, the result is:

```text
"t3.small"
```

I tested this using `terraform console`.

The tests returned:

```text
dev  -> "t3.micro"
prod -> "t3.small"
```

I also tested the production condition directly:

```hcl
"prod" == "prod" ? "t3.small" : "t3.micro"
```

Result:

```text
"t3.small"
```

---

# 10. Five Functions I Found Useful

### 1. upper()

Converts text into uppercase.

```hcl
upper("terraform")
```

### 2. join()

Joins values from a list using a separator.

```hcl
join("-", ["terraform", "aws", "dev"])
```

### 3. length()

Returns the number of items in a list or collection.

```hcl
length(["a", "b", "c"])
```

### 4. lookup()

Gets a value from a map using a key.

```hcl
lookup({dev = "t2.micro", prod = "t3.small"}, "prod")
```

### 5. cidrsubnet()

Creates a subnet CIDR from a larger network.

```hcl
cidrsubnet("10.0.0.0/16", 8, 1)
```

---

# 11. Variable vs Local vs Output vs Data

| Feature | What I understood |
|---|---|
| Variable | Takes input values |
| Local | Stores values or expressions that I reuse |
| Output | Displays useful values after apply |
| Data Source | Reads information from AWS |

A simple way I remember it:

```text
Variable → Input
Local    → Reuse
Data     → Read
Output   → Show
```

---

# 12. Verification in AWS

I checked the resources in the AWS Console.

## VPC

The VPC was created with the name:

```text
terraweek-dev-vpc
```

CIDR:

```text
10.0.0.0/16
```

## Subnet

The subnet was created with:

```text
terraweek-dev-subnet
```

CIDR:

```text
10.0.1.0/24
```

Availability Zone:

```text
ap-south-1a
```

## Security Group

The security group was created as:

```text
terraweek-dev-sg
```

The inbound rules were created for:

```text
22
80
443
```

## EC2 Instance

The EC2 instance was created with:

```text
Name: terraweek-dev-server
Instance Type: t2.micro
State: Running
```

The public IP was:

```text
13.201.47.201
```

---

