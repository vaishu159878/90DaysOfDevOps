# Day 50 – Kubernetes Architecture and Cluster Setup

## Objective

Today I started my Kubernetes journey. I learned Kubernetes architecture,
installed kubectl, created a local cluster using kind and explored the
Kubernetes system components.

## 1. Kubernetes Story

Docker is useful for running containers, but managing many containers across
multiple servers becomes difficult. Kubernetes solves this problem by
providing container orchestration, scheduling, networking, scaling and
desired-state management.

Kubernetes was originally created at Google and was influenced by systems
such as Borg.

## 2. Kubernetes Architecture

![Kubernetes architecture](./kubenetes-architecture.jpeg)

### Control Plane

- **API Server** – Entry point for Kubernetes requests
- **etcd** – Stores cluster state
- **Scheduler** – Decides where Pods should run
- **Controller Manager** – Maintains the desired state

### Worker Node

- **kubelet** – Manages Pods on the node
- **kube-proxy** – Handles networking
- **Container Runtime** – Runs containers

### Architecture

## 3. kubectl apply Flow
kubectl
   ↓
API Server
   ↓
etcd
   ↓
Scheduler
   ↓
kubelet
   ↓
Container Runtime
   ↓
Pod

If the API Server goes down, new kubectl requests cannot be processed.

If a worker node goes down, Kubernetes detects the failure and controllers
can work to maintain the desired state.

## 4. kubectl Installation

I installed kubectl on my Ubuntu EC2 instance.

kubectl version --client

## 5. Cluster Setup

I chose kind (Kubernetes in Docker) because I already had Docker
installed and wanted to practice Kubernetes using Docker.

kind create cluster --name devops-cluster

I verified the cluster:

kubectl cluster-info
kubectl get nodes


## 6. Exploring the Cluster

kubectl get namespaces
kubectl get pods -A
kubectl get pods -n kube-system

I explored the Kubernetes system components such as:

Component	Purpose
etcd	Stores cluster state
kube-apiserver	Handles API requests
kube-scheduler	Selects nodes for Pods
kube-controller-manager	Maintains desired state
kube-proxy	Handles networking
coredns	Provides DNS

## 7. Cluster Lifecycle

I practiced deleting and recreating the cluster.

kind delete cluster --name devops-cluster
kind create cluster --name devops-cluster
kubectl get nodes

The recreated cluster showed the control-plane node as Ready.

During the practice, I also noticed scheduler and controller-manager errors
and learned that checking system Pods is important for troubleshooting.

## 8. Kubernetes Context & Kubeconfig
kubectl config current-context
kubectl config get-contexts
kubectl config view

Current context:

kind-devops-cluster

Default kubeconfig location:

~/.kube/config

Kubeconfig contains information about clusters, users and contexts used by
kubectl.

