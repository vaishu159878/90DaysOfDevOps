# Day 65 - Terraform Modules

## What I learned

Today I learned about Terraform modules and how they help to make Terraform code reusable.

---

## 1. Terraform Module Structure

I created the following structure:

```text
terraform-modules/
├── main.tf
├── variables.tf
├── outputs.tf
├── providers.tf
└── modules/
    ├── ec2-instance/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    └── security-group/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

### Root Module

The root module is the main Terraform project where I run commands like:

```bash
terraform init
terraform plan
terraform apply
terraform destroy
```

The root module calls the child modules.

### Child Module

A child module is a reusable Terraform configuration. In my project I created two local child modules:

```text
modules/ec2-instance
modules/security-group
```

The root module passes values to these modules and uses their outputs.

I understood modules similar to functions in programming. I define the infrastructure once and then call it whenever I need it.

---

## 2. Custom EC2 Module

I created an EC2 module under:

```text
modules/ec2-instance/
```

### variables.tf

```hcl
variable "ami_id" {
  description = "AMI ID for the EC2 instance"
  type        = string
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "subnet_id" {
  description = "Subnet ID where the instance will be launched"
  type        = string
}

variable "security_group_ids" {
  description = "Security group IDs"
  type        = list(string)
}

variable "instance_name" {
  description = "Name tag for the EC2 instance"
  type        = string
}

variable "tags" {
  description = "Additional tags"
  type        = map(string)
  default     = {}
}
```

### main.tf

```hcl
resource "aws_instance" "this" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.security_group_ids

  tags = merge(
    var.tags,
    {
      Name = var.instance_name
    }
  )
}
```

### outputs.tf

```hcl
output "instance_id" {
  description = "EC2 instance ID"
  value       = aws_instance.this.id
}

output "public_ip" {
  description = "Public IP address"
  value       = aws_instance.this.public_ip
}

output "private_ip" {
  description = "Private IP address"
  value       = aws_instance.this.private_ip
}
```

The important part here is that the module does not have a fixed instance name. I pass the name from the root module.

---

## 3. Custom Security Group Module

I created another module:

```text
modules/security-group/
```

This module creates a security group and generates ingress rules based on a list of ports.

### variables.tf

```hcl
variable "vpc_id" {
  description = "VPC ID"
  type        = string
}

variable "sg_name" {
  description = "Security group name"
  type        = string
}

variable "ingress_ports" {
  description = "Ports to allow inbound"
  type        = list(number)
  default     = [22, 80]
}

variable "tags" {
  description = "Additional tags"
  type        = map(string)
  default     = {}
}
```

### main.tf

```hcl
resource "aws_security_group" "this" {
  name        = var.sg_name
  description = "Security group managed by Terraform module"
  vpc_id      = var.vpc_id

  dynamic "ingress" {
    for_each = var.ingress_ports

    content {
      description = "Allow TCP port ${ingress.value}"
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  egress {
    description = "Allow all outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = var.sg_name
    }
  )
}
```

### outputs.tf

```hcl
output "sg_id" {
  description = "Security group ID"
  value       = aws_security_group.this.id
}
```

### Dynamic Block

This was my first time using a `dynamic` block.

I used:

```hcl
dynamic "ingress" {
  for_each = var.ingress_ports

  content {
    ...
  }
}
```

I passed:

```hcl
ingress_ports = [22, 80, 443]
```

Terraform then created ingress rules for:

- SSH - 22
- HTTP - 80
- HTTPS - 443

This is useful because I don't have to write the same ingress block three times.

---

## 4. Calling the Custom Modules

I called the Security Group module from the root module:

```hcl
module "web_sg" {
  source = "./modules/security-group"

  vpc_id        = module.vpc.vpc_id
  sg_name       = "terraweek-web-sg"
  ingress_ports = [22, 80, 443]

  tags = local.common_tags
}
```

Then I used the Security Group output in the EC2 module.

### Web Server

```hcl
module "web_server" {
  source = "./modules/ec2-instance"

  ami_id             = data.aws_ami.amazon_linux.id
  instance_type      = "t3.micro"
  subnet_id          = module.vpc.public_subnets[0]
  security_group_ids = [module.web_sg.sg_id]

  instance_name = "terraweek-web"

  tags = local.common_tags
}
```

### API Server

```hcl
module "api_server" {
  source = "./modules/ec2-instance"

  ami_id             = data.aws_ami.amazon_linux.id
  instance_type      = "t3.micro"
  subnet_id          = module.vpc.public_subnets[0]
  security_group_ids = [module.web_sg.sg_id]

  instance_name = "terraweek-api"

  tags = local.common_tags
}
```

The main thing I wanted to understand was that I didn't have to create two separate EC2 resource blocks.

I reused the same module twice:

```text
              EC2 Module
              /        \
             /          \
     Web Server        API Server
```

Both instances use the same module but have different names.

---

## 5. VPC Registry Module

Instead of creating every VPC resource manually, I used the public Terraform Registry VPC module.

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "terraweek-vpc"
  cidr = "10.0.0.0/16"

  azs = [
    "ap-south-1a",
    "ap-south-1b"
  ]

  public_subnets = [
    "10.0.1.0/24",
    "10.0.2.0/24"
  ]

  private_subnets = [
    "10.0.3.0/24",
    "10.0.4.0/24"
  ]

  enable_nat_gateway   = false
  enable_dns_hostnames = true

  tags = local.common_tags
}
```

I then used outputs from the VPC module:

```hcl
module.vpc.vpc_id
```

and:

```hcl
module.vpc.public_subnets[0]
```

This allowed my custom modules to use the infrastructure created by the registry module.

---

## 6. Terraform Apply Result

I ran:

```bash
terraform init
terraform plan
terraform apply
```

Terraform successfully created the infrastructure.

The final result was:

```text
Apply complete! Resources: 20 added, 0 changed, 0 destroyed.
```

So in this run Terraform created **20 resources**.

The VPC module created the main networking resources including:

- VPC
- Internet Gateway
- 2 public subnets
- 2 private subnets
- Public route table
- 2 private route tables
- Route table associations
- Default network ACL
- Default route table
- Default security group

The custom modules created:

- 2 EC2 instances
- 1 custom security group

---

## 7. Terraform Outputs

My Terraform outputs after apply were:

```text
api_server_id     = "i-0d3f24307d78f652b"
api_server_ip     = ""
security_group_id = "sg-09edc246b2b5c5cdd"
vpc_id            = "vpc-0649cd6162a654fa4"
web_server_id     = "i-0159402164b4015c9"
web_server_ip     = ""
```

The two EC2 instances were created successfully:

```text
Web Server: terraweek-web
API Server: terraweek-api
```

The public IP outputs were empty in this run, so I checked the EC2 console/network configuration separately rather than assuming that a public IP was assigned.

---

## 8. Terraform State

I ran:

```bash
terraform state list
```


This helped me understand how Terraform keeps track of resources inside modules.

For example:

```text
module.web_server.aws_instance.this
```

means the EC2 resource belongs to the `web_server` module.

---

## 9. Registry Module Download

I checked:

```bash
ls -la .terraform/modules/
```

I found:

```text
modules.json
vpc/
```

So Terraform downloaded the registry VPC module into the `.terraform/modules/` directory.

The local custom modules are referenced directly from:

```text
./modules/ec2-instance
./modules/security-group
```

---

## 10. Hand-written VPC vs Registry Module

### Hand-written VPC

When writing a VPC manually, I have to create and maintain resources such as:

```text
VPC
Internet Gateway
Subnets
Route Tables
Routes
Route Table Associations
Default networking resources
```

This gives me more direct control, but it also means more Terraform code to maintain.

### Registry VPC Module

With the registry module, the root configuration is much smaller:

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  ...
}
```

The module handles the underlying AWS resources for me.

In my actual run, the VPC module was part of a plan that created **20 resources in total**, including the two EC2 instances and the custom security group.

So the `20` is the total Terraform plan result, not only the number of resources created by the VPC module.

---

## 11. Module Versioning

I learned that registry modules can be version constrained.

For example:

### Exact version

```hcl
version = "5.1.0"
```

This uses exactly version 5.1.0.

### Compatible 5.x versions

```hcl
version = "~> 5.0"
```

This allows compatible releases in the 5.x version range.

### Version range

```hcl
version = ">= 5.0, < 6.0"
```

This allows versions from 5.0 up to, but not including, 6.0.

I can check for newer compatible module versions with:

```bash
terraform init -upgrade
```

---

