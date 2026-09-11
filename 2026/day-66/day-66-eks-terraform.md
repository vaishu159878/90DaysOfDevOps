# Day 66 --- Provision AWS EKS with Terraform

## Overview

For Day 66 of my #90DaysOfDevOps journey, I worked on provisioning an
**Amazon EKS cluster using Terraform** and then deployed an Nginx
application on Kubernetes.

The main goal was to understand how Terraform can be used to create AWS
infrastructure and how Kubernetes can be used to deploy applications on
top of that infrastructure.

This was also a good troubleshooting exercise because I faced a couple
of issues during the setup and had to fix them before the deployment
worked successfully.

------------------------------------------------------------------------

## What I Built

The final setup looked like this:

``` text
Terraform
    |
    v
AWS VPC
    |
    +-- Public Subnets
    +-- Private Subnets
    +-- NAT Gateway
    |
    v
Amazon EKS
    |
    +-- 2 Managed Worker Nodes (t3.small)
    |
    v
Kubernetes
    |
    +-- Nginx Deployment
    |      |
    |      +-- 3 Replicas
    |
    +-- LoadBalancer Service
              |
              v
        AWS Load Balancer
              |
              v
        Nginx Welcome Page
```

------------------------------------------------------------------------

## Technologies Used

-   Terraform
-   AWS
-   Amazon EKS
-   Amazon VPC
-   Kubernetes
-   kubectl
-   Nginx
-   AWS CLI
-   Git & GitHub

------------------------------------------------------------------------

## Project Structure

``` text
terraform-eks/
├── providers.tf
├── vpc.tf
├── eks.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
└── k8s/
    └── nginx-deployment.yaml
```

------------------------------------------------------------------------

## Terraform Configuration

I used Terraform modules for the AWS networking and EKS setup.

### VPC

The VPC configuration included:

-   VPC CIDR: `10.0.0.0/16`
-   2 Availability Zones
-   2 public subnets
-   2 private subnets
-   Single NAT Gateway
-   DNS hostnames enabled
-   DNS support enabled
-   Kubernetes subnet tags for load balancers

### EKS

The EKS cluster configuration included:

-   Cluster name: `terraweek-eks`
-   Kubernetes version: `1.31`
-   Managed node group
-   2 desired worker nodes
-   Minimum nodes: 1
-   Maximum nodes: 3
-   Instance type: `t3.small`
-   Private subnets for worker nodes
-   Public EKS API endpoint

------------------------------------------------------------------------

## Steps I Followed

### 1. Initialize Terraform

``` bash
terraform init
```

### 2. Validate the configuration

``` bash
terraform validate
```

### 3. Review the infrastructure

``` bash
terraform plan
```

### 4. Provision AWS infrastructure

``` bash
terraform apply
```

After confirming with `yes`, Terraform created the VPC, EKS cluster,
managed node group and supporting AWS resources.

------------------------------------------------------------------------

## Troubleshooting

### Issue 1 --- Node group failed with `t3.medium`

Initially I used:

``` text
t3.medium
```

The managed node group failed because the instance type was not eligible
for the Free Tier restriction for my AWS account/region.

I checked the available Free Tier eligible instance types using:

``` bash
aws ec2 describe-instance-types \
  --region us-east-1 \
  --filters "Name=free-tier-eligible,Values=true" \
  --query 'InstanceTypes[].InstanceType' \
  --output table
```

I selected:

``` text
t3.small
```

Then I updated the Terraform configuration and recreated the failed node
group.

The next Terraform plan showed:

``` text
Plan: 1 to add, 0 to change, 0 to destroy.
```

The node group was then created successfully.

------------------------------------------------------------------------

### Issue 2 --- kubectl authentication error

After the EKS cluster was created, AWS CLI authentication was working:

``` bash
aws sts get-caller-identity
```

and I was also able to generate an EKS token:

``` bash
aws eks get-token \
  --cluster-name terraweek-eks \
  --region us-east-1
```

However, `kubectl get nodes` initially returned:

``` text
the server has asked for the client to provide credentials
```

I checked the EKS authentication mode:

``` bash
aws eks describe-cluster \
  --name terraweek-eks \
  --region us-east-1 \
  --query 'cluster.accessConfig'
```

The cluster was using:

``` text
API_AND_CONFIG_MAP
```

My IAM user did not have an EKS access entry.

I fixed this by enabling cluster creator admin permissions in the
Terraform EKS module:

``` hcl
enable_cluster_creator_admin_permissions = true
```

After applying the change, my IAM user was added as an EKS access entry
and `kubectl` started working.

------------------------------------------------------------------------

## Connecting kubectl to EKS

I configured the Kubernetes context with:

``` bash
aws eks update-kubeconfig \
  --name terraweek-eks \
  --region us-east-1
```

Then I verified the worker nodes:

``` bash
kubectl get nodes
```

Result:

``` text
NAME                          STATUS   ROLES    AGE   VERSION
ip-10-0-11-28.ec2.internal    Ready    <none>   ...   v1.31.13-eks-ecaa3a6
ip-10-0-12-243.ec2.internal   Ready    <none>   ...   v1.31.13-eks-ecaa3a6
```

Both worker nodes were in the `Ready` state.

------------------------------------------------------------------------

## Kubernetes System Pods

I also checked the Kubernetes system pods:

``` bash
kubectl get pods -A
```

The important components were running successfully:

-   CoreDNS
-   AWS VPC CNI (`aws-node`)
-   kube-proxy

------------------------------------------------------------------------

## Deploying Nginx

I created a Kubernetes manifest containing:

-   Deployment
-   3 Nginx replicas
-   LoadBalancer Service

I deployed it with:

``` bash
kubectl apply -f k8s/nginx-deployment.yaml
```

Then checked the deployment:

``` bash
kubectl get deployment
```

Result:

``` text
NAME              READY   UP-TO-DATE   AVAILABLE
nginx-terraweek   3/3     3            3
```

I also checked the pods:

``` bash
kubectl get pods -o wide
```

All 3 Nginx pods were running successfully.

------------------------------------------------------------------------

## Exposing Nginx

The service was created as a Kubernetes LoadBalancer:

``` bash
kubectl get svc nginx-service
```

The service received an AWS Load Balancer hostname.

I tested the application using:

``` bash
curl http://$(kubectl get svc nginx-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
```

The response was the Nginx Welcome page.

I also opened the Load Balancer address in a browser and verified that
the Nginx Welcome Page was accessible.

------------------------------------------------------------------------

## Verification

The final working setup was:

``` text
EKS Cluster       → terraweek-eks
Kubernetes        → 1.31
Worker Nodes      → 2
Instance Type     → t3.small
Nginx Replicas    → 3
Service Type      → LoadBalancer
Application       → Nginx Welcome Page
```

Terraform state contained:

``` text
69 resources
```

during the completed deployment.

------------------------------------------------------------------------

## Cleanup

After taking the required screenshots and verifying the application, I
removed the Kubernetes resources first:

``` bash
kubectl delete -f k8s/nginx-deployment.yaml
```

Then I destroyed the AWS infrastructure using:

``` bash
terraform destroy
```

Terraform reported:

``` text
Destroy complete! Resources: 57 destroyed.
```

Finally, I verified the Terraform state:

``` bash
terraform state list
```

The state was empty.

I also verified the EKS clusters:

``` bash
aws eks list-clusters --region us-east-1
```

Result:

``` json
{
    "clusters": []
}
```

This confirmed that the EKS cluster and the Terraform-managed
infrastructure had been cleaned up successfully.

------------------------------------------------------------------------

## What I Learned

This lab helped me understand how different parts of AWS and Kubernetes
work together.

### My key takeaways:

1.  **Terraform modules** make it easier to build reusable AWS
    infrastructure.
2.  **EKS** provides the managed Kubernetes control plane while worker
    nodes run the workloads.
3.  Kubernetes worker nodes can run in private subnets while the
    application can be exposed using a LoadBalancer.
4.  **IAM and Kubernetes access are closely connected** when working
    with EKS.
5.  Reading the actual AWS error messages helped me identify the
    instance type problem.
6.  `kubectl` is the main tool I used to verify and manage workloads in
    the EKS cluster.
7.  Cleaning up cloud resources is important to avoid unnecessary AWS
    charges.

The biggest learning for me was that getting infrastructure deployed is
only one part of DevOps. Troubleshooting authentication, networking,
compute capacity and application access is equally important.

------------------------------------------------------------------------

