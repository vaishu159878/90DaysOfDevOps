# Amazon EKS with Terraform

Today I learned how to create and manage an Amazon EKS cluster using Terraform and deploy my BankApp application on Kubernetes.

This was one of the most practical days of my #90DaysOfDevOps journey because I worked with AWS, Terraform, Kubernetes, EBS storage, HPA, ArgoCD and Git.

## What I Learned

- What Amazon EKS is
- How EKS managed control plane works
- How worker nodes connect to an EKS cluster
- How to create EKS infrastructure using Terraform
- How AWS VPC and subnets are used with EKS
- How EBS CSI provides persistent storage
- How Metrics Server and HPA work
- How to deploy an application on Kubernetes
- How ArgoCD fits into GitOps
- How to troubleshoot AWS and Kubernetes errors

## Architecture

```text
AWS
 |
 VPC
 |
 EKS Cluster
 |
 +-- Node 1
 +-- Node 2
 +-- Node 3
 |
 Kubernetes Workloads
 |
 +-- BankApp
 +-- MySQL ---- EBS CSI Persistent Storage
 +-- Ollama
 |
 AWS Load Balancer
 |
 BankApp in Browser
```

## EKS Node Configuration

Initially, the worker node instance type was `t3.medium`.

During deployment, AWS returned an instance eligibility error for my account. I checked the available instance types and changed the worker nodes to `t3.small`.

This allowed the EKS node group to be created successfully.

## Create the EKS Cluster

I created the infrastructure using Terraform and then verified that the EKS cluster was active.

I also configured `kubectl` to connect to the cluster and verified that all three worker nodes were ready.

## Kubernetes System Components

The following components were running successfully:

- AWS VPC CNI
- CoreDNS
- kube-proxy
- EBS CSI Driver
- EKS Pod Identity Agent
- Metrics Server

## Persistent Storage

I configured AWS EBS CSI to provide persistent storage for Kubernetes workloads.

The application uses `gp3` storage with persistent volume claims for:

- MySQL
- Ollama

Both PVCs were successfully bound.

## Deploy BankApp

I deployed:

- BankApp
- MySQL
- Ollama

All application components were successfully deployed and running.

## Ollama Scheduling Issue

While deploying Ollama, the pod initially remained in `Pending` state.

After checking the pod events, I found that the worker nodes did not have enough memory for the original resource requirements.

I reduced the Ollama CPU and memory requests/limits and deployed it again.

After the change, Ollama was running successfully.

This was a good lesson about Kubernetes resource requests, limits and pod scheduling.

## Metrics Server and HPA

I verified that Metrics Server was running and that Kubernetes was receiving CPU and memory metrics.

I configured HPA for BankApp with:

- Minimum replicas: 2
- Maximum replicas: 4
- CPU target: 70%

At the time of verification, CPU usage was low, so the application was running with 2 replicas.

## Expose BankApp

Initially, the BankApp service was configured as a ClusterIP service.

I wanted to access the application from my browser, so I exposed it through an AWS Load Balancer.

The application was then accessible from the browser.

## ArgoCD

I installed ArgoCD in the EKS cluster and verified that all ArgoCD components were running successfully.

The ArgoCD dashboard was also accessible through the Load Balancer.

## Troubleshooting

This day was not just about following commands.

I faced several problems during the deployment.

### EC2 Instance Eligibility

The initial `t3.medium` worker nodes could not be created because of the AWS account's instance eligibility.

**Solution:** Changed the worker nodes to `t3.small`.

### Missing IAM Role

During the Terraform deployment, an EKS IAM role was missing.

I checked the AWS IAM resources and recreated the required role so Terraform could continue managing the cluster.

### Ollama Pod Pending

Ollama could not be scheduled because the worker nodes did not have enough memory.

**Solution:** Reduced the CPU and memory requests/limits for Ollama.

### Load Balancer Configuration

The Kubernetes service initially used client IP session affinity, which was not supported by the AWS Load Balancer configuration.

**Solution:** Removed the unsupported session affinity configuration and exposed the service successfully.

## What I Verified

```text
EKS Cluster       → ACTIVE
Worker Nodes      → 3/3 Ready
BankApp           → Running
MySQL             → Running
Ollama            → Running
EBS CSI           → Running
Metrics Server    → Running
HPA               → Working
ArgoCD            → Running
Load Balancer     → Working
Git Repository    → Clean
```

## What I Learned

The biggest lesson from today was troubleshooting.

Things did not work perfectly on the first attempt.

I learned how to:

- Read Terraform errors
- Check AWS resources
- Troubleshoot IAM issues
- Understand Kubernetes pod scheduling
- Check CPU and memory usage
- Work with persistent volumes
- Configure Kubernetes services
- Understand how AWS Load Balancers work with Kubernetes
- Verify an EKS environment after deployment

Instead of just running commands, I started understanding **why something failed and how to fix it**.

## Tools Used

- AWS
- Amazon EKS
- Terraform
- Kubernetes
- Docker
- MySQL
- Ollama
- ArgoCD
- Git
- GitHub
- Linux

## Final Result

I successfully created an Amazon EKS cluster using Terraform, deployed my BankApp and its supporting services, configured persistent storage and autoscaling, installed ArgoCD, and exposed the application through an AWS Load Balancer.

This was a challenging but very useful hands-on experience.

**Learning by building, breaking, debugging and fixing. 🚀**

