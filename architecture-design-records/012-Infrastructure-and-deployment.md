# Infrastructure Resource and Deployment Design

This document covers resources used in the infrastructure and their deployment methods. It also covers the reasoning behind the deployment strategy. Standard deployment practices were adopted. (Continious integration and Continous deployment)

## General Setup

**Infrastructure as Code**

* Terraform used as Infrastructure as code to deploy infrastructure resource on AWS
* Full instruction of setup can be found in the readme.md in the infra repo. 
* location [here](https://github.com/nationalarchives/da-ayr-terraform-infra)

**Code Repository**
* Github is used as code repository for infrastructure and application repository. See below breakdown:

    - Infrastructure [Check](https://github.com/nationalarchives/da-ayr-terraform-infra)
    - Keycloak Application [Check](https://github.com/nationalarchives/da-ayr-auth-server)
    - Web Client Application [Check](https://github.com/nationalarchives/da-ayr-webapp)
    - Lambda Functions [Check](https://github.com/nationalarchives/da-ayr-lambdas)

**Deployment Pipelines**

* Github-Actions pipelnes used for buildng artefacts and deploying the infrastructure and application
* Github action are stored [here](https://github.com/nationalarchives/da-ayr-github-actions)

**Artefact Repository**
* Artefacts are pushed and stored in s3 bucket with versioning enabled

## Infrastructure Resource

Core Resource needed in the infrastructore are:
* Virtual Private Cloud: Host the application on the Cloud in AWS using VPC, Subnets etc
* Data Storage/Source: Relational Databases for the service i.e  Posgres, S3 Buckets
* Server/Cluster: Docker containers on ECS Cluster for running the applications
* Serverless Resources: Lambda functins, API gateway for ingesting and transforming data


## Deployment Design

**Main Deployment:**

This is the deployment of the base infrastructure, this include 
   - vpc
   - subnets 
   - internet gateway
   - Nat Gateway 
   - databases, 
   - ACLs and security groups
   - Elastic IP Addresses
   - Application load balancer
   - ECR /ECS base resource
   - Setup resource for lambda and API gateway

**Child Deployment:** 

This is the deployments of the applications and serverless resources i.e lambda and ecr/ecs. These resource are likely to change more frequently. Child deployment can be done independent of the Main as long as the Main already exist.

   - Lambda functions
   - API Gateway
   - ECR images running on ECS Cluster

Below is a diagram of the deployment strategy used that allows a base deployment but also independent application deployment

![Infrastructure and deployment](images/deployment-diagram.png)


## Findings and Adobtions

**Serverless Design (Lambda API Gateway)**
The service is performing multiple tasks that are intermittent, i.e ingesting and transforming data. A serverless approach was adopted to maximise resource and minimise cost. (Lambda functions, api gateway)

**Containerization (ECR/ECS)**
A reliable service that is resourceful and can scale seamlessly was the idea behind adopting ecr and ecs cluster to host and run the authentication and search service.

**Search and Indexing**
Tested a few search applications but ended up using AWS Opensearch. An evaluation of other the other in comparisn can be found [here](008-search-engine-options.md) 
