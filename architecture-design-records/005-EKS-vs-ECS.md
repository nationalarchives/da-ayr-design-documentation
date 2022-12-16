# EKS vs ECS

Date: 16-12-2022

## Status

Accepted

## Context

### EKS (Elastic Kubernetes Service)

Amazon Elastic Kubernetes Service (Amazon EKS) makes it easy to deploy, manage, and scale containerized applications using Kubernetes on Amazon Web Services. Amazon EKS runs the Kubernetes management infrastructure for you across multiple Amazon Web Services availability zones to eliminate a single point of failure. Amazon EKS is certified Kubernetes conformant so you can use existing tooling and plugins from partners and the Kubernetes community. Applications running on any standard Kubernetes environment are fully compatible and can be easily migrated to Amazon EKS.

#### Benefits
- You don’t have to install, operate, and maintain your own Kubernetes control plane.
- EKS allows you to easily run tooling and plugins developed by the Kubernetes open-source community. 
- EKS automates load distribution and parallel processing better than any DevOps engineer could.
- Your Kubernetes assets integrate seamlessly with AWS services if you use EKS.
- EKS uses VPC networking.
- Any application running on EKS is compatible with one running in your existing Kubernetes environment. You can migrate to EKS without applying any changes to code.
- Supports EC2 spot instances using managed node groups that follow spot best practices and allow some pretty great cost savings. 

### ECS (Elastic Container Service)

ECS is a convenient way to run Docker containers in AWS platform. AWS refers ECS to "ECS on EC2" or "Fargate" interchangeably.

Fargate is a serverless technology, where the Cluster management and orchestration of containers are supported; this may be suitable when a large no. of docker containers needs to be managed.
Tapestry, Loom and CyberFirst however, have relatively small no. of micro-services, so Fargate/Kubernetes is not used. In this document, we refer ECS to "ECS on EC2" only.

The benefit of ECS is the relative simplicity, can be cost-efficient and works well when there are few no. (< 5) of micro services to run. 

However, managing ECS requires additional work such as provisioning the Docker host machines (termed as ECS clusters) and the orchestration of the docker containers.

Additional info can be found here https://aws.amazon.com/ecs/features/

## Decision

Since the cluster will run only the Django application, the recommendation is to use the ECS cluster with Fargate which will be easier to maintain.

## Consequences

N/A
