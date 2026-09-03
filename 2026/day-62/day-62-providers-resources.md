# Day 62 - Providers, Resources and Dependencies

## Introduction

Today I worked on Terraform with AWS and learned how Terraform manages providers, resources and dependencies.

Instead of creating everything manually from the AWS console, I used Terraform to create the infrastructure.

For this task I created a VPC, public subnet, Internet Gateway, route table, security group, EC2 instance and an S3 bucket.

---

## Task 1 - AWS Provider

First I created a project directory:

```bash
mkdir terraform-aws-infra
cd terraform-aws-infra
```

I created a `providers.tf` file:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}
```

Then I ran:

```bash
terraform init
```

Terraform installed:

```text
hashicorp/aws v5.100.0
```


### What does `~> 5.0` mean?

`~> 5.0` allows Terraform to use compatible versions from the 5.x series, but it does not allow version 6.x.

For example:

- `~> 5.0` - allows compatible 5.x versions
- `>= 5.0` - allows 5.0 and newer versions
- `= 5.0.0` - allows only exactly 5.0.0

### `.terraform.lock.hcl`

Terraform automatically created `.terraform.lock.hcl`.

It records the selected provider version and checksums. This helps Terraform use the same provider version in future runs and on other machines.

I will commit this file to Git, but I will not commit the `.terraform` directory.

---

## Task 2 - Build a VPC from Scratch

I created the following AWS resources:

- VPC
- Public Subnet
- Internet Gateway
- Route Table
- Route Table Association

My VPC CIDR is:

```text
10.0.0.0/16
```

My public subnet CIDR is:

```text
10.0.1.0/24
```

### My Terraform configuration

```hcl
# VPC
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "TerraWeek-VPC"
  }
}

# Public Subnet
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true

  tags = {
    Name = "TerraWeek-Public-Subnet"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "TerraWeek-IGW"
  }
}

# Public Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "TerraWeek-Public-Route-Table"
  }
}

# Route Table Association
resource "aws_route_table_association" "public" {
  route_table_id = aws_route_table.public.id
  subnet_id      = aws_subnet.public.id
}
```

I checked the configuration using:

```bash
terraform validate
```

The result was:

```text
Success! The configuration is valid.
```

---

## Task 3 - Understanding Implicit Dependencies

One of the main things I learned today was how Terraform understands dependencies automatically.

For example, in the subnet I used:

```hcl
vpc_id = aws_vpc.main.id
```

Because the subnet needs the VPC ID, Terraform knows that the VPC has to be created first.

The same thing happens with the Internet Gateway:

```hcl
vpc_id = aws_vpc.main.id
```

The route table also depends on the VPC and Internet Gateway:

```hcl
vpc_id     = aws_vpc.main.id
gateway_id = aws_internet_gateway.main.id
```

The route table association depends on both the subnet and route table:

```hcl
route_table_id = aws_route_table.public.id
subnet_id      = aws_subnet.public.id
```

### Dependencies I found

```text
VPC
├── Subnet
├── Internet Gateway
├── Route Table
└── Security Group

Subnet + Security Group
└── EC2

Internet Gateway
└── Route Table

Subnet + Route Table
└── Route Table Association
```

Terraform does not simply execute my resources from top to bottom. It looks at these relationships and creates resources in the required order.

If I tried to create a subnet without an existing VPC, AWS would not be able to create the subnet because a subnet must belong to a VPC.

---

## Task 4 - Security Group and EC2

Next I added a security group.

It allows:

- SSH - port 22
- HTTP - port 80
- All outbound traffic

```hcl
resource "aws_security_group" "main" {
  name        = "TerraWeek-SG"
  description = "Allow SSH and HTTP traffic"
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "TerraWeek-SG"
  }
}
```

For the EC2 instance, I used an Amazon Linux 2 AMI from the Mumbai region.

I used a data source to find the AMI:

```hcl
data "aws_ami" "amazon_linux_2" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}
```

Then I created the EC2 instance:

```hcl
resource "aws_instance" "main" {
  ami                         = data.aws_ami.amazon_linux_2.id
  instance_type               = "t2.micro"
  subnet_id                   = aws_subnet.public.id
  vpc_security_group_ids      = [aws_security_group.main.id]
  associate_public_ip_address = true

  lifecycle {
    create_before_destroy = true
  }

  tags = {
    Name = "TerraWeek-Server"
  }
}
```

Terraform successfully created the EC2 instance.


The EC2 instance received this public IP during my test:

```text
13.203.44.183
```

---

## Task 5 - Explicit Dependency with depends_on

I also created an S3 bucket for application logs.

The bucket does not directly use anything from the EC2 instance, so Terraform would not automatically create a dependency between them.

I added:

```hcl
resource "aws_s3_bucket" "logs" {
  bucket_prefix = "terraweek-app-logs-"

  depends_on = [
    aws_instance.main
  ]

  tags = {
    Name = "TerraWeek-App-Logs"
  }
}
```

This means Terraform will create the S3 bucket only after the EC2 instance has been created.

The S3 bucket was created successfully.


### Implicit vs Explicit Dependency

**Implicit dependency:**

Terraform automatically detects the dependency from a resource reference.

Example:

```hcl
vpc_id = aws_vpc.main.id
```

Terraform understands:

```text
VPC -> Subnet
```

**Explicit dependency:**

I manually tell Terraform about the dependency using `depends_on`.

Example:

```hcl
depends_on = [
  aws_instance.main
]
```

Terraform understands:

```text
EC2 -> S3 Bucket
```

### When would I use `depends_on`?

I would use `depends_on` when Terraform cannot automatically detect a dependency.

For example:

1. When a resource depends on another resource being completely created even though there is no direct reference.
2. When a resource depends on some initialization or side effect that Terraform cannot understand automatically.

I would prefer implicit dependencies whenever possible because they make the configuration easier to understand.

---

## Task 6 - Lifecycle Rules

I used this lifecycle configuration for the EC2 instance:

```hcl
lifecycle {
  create_before_destroy = true
}
```

### 1. create_before_destroy

```hcl
create_before_destroy = true
```

Terraform creates the replacement resource before destroying the old resource.

I would use this when I want to reduce downtime during resource replacement.

### 2. prevent_destroy

```hcl
prevent_destroy = true
```

This prevents Terraform from destroying the resource.

I would use this for important resources where accidental deletion could cause a serious problem, such as a production database.

### 3. ignore_changes

```hcl
ignore_changes = [
  tags
]
```

This tells Terraform to ignore changes to selected attributes.

I would use this when another system or AWS service is allowed to modify an attribute and I don't want Terraform to continuously overwrite that change.

---






