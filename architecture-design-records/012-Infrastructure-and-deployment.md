# Infrastructure Resource and Deployment Design

This document covers resources used in the infrastructure and their deployment methods. Standard deployment practices were adopted. (Infrastructure as code,Continious integration and deployment)

## General Setup

1. Terraform is used to build the infrastructure resource on AWS
2. Github is used as code repository
3. Github-Actions is used to build and deploy the infrastructure and application
4. S3 Bucket is used for built artefact 

The two KeyCloak realms have different responsibilities for managing authentication and authorisation.

TDR realm provides:
1. Primary user account entry.
2. User credentials

AYR realm provides:
1. Secondary account entry.
2. User group memberships and role assignments.

## User onboarding and login process

![Infrastructure and deployment](images/deployment-diagram.png)

To add a user to AYR under this arrangement, the following steps would be undertaken.





1) what problem we were trying to solve, 
PROJECT

INFRA RESOURCES NEEDED

Core Resource needed in the infrastructore are:

Virtual Private Cloud
Host the application on the Cloud in AWS using VPC, Subnets etc

Data Storage/Source
Relational Databases Posgres, S3 Buckets

Server/Cluster
Docker containers running on ECS Cluster

Serverless Resources
Lambda functins, API gateway.


2) how we approached it, 
APPROACH

We are using AWS to build and host the Service.

Actions
- Terraform is used to build the infrastructure resource on AWS
- Github is used as code repository
- Github-Actions is used to build and deploy the infrastructure and application
- S3 Bucket is used for built artefact 

Deployment
There are Main and child type deployments. 
Main: deployment of the base infrastructure, this include vpc, subnets internet gateway database etc
Child: deployments of the applications and serverless resources i.e lambda and ecr/ecs. 
Child deployment can be done independent of the Main as long as the Main already exist

Main deployment
Deploying VPC, Subnets, Databases, s3 buckets, log files

Child deployments
Deploying lambdas and ecr/ecs


See Diagram below



3) how it went - what worked / didn't work. Ideally done today but Monday if not today, please.

FINDINGS 
Research shows that AWS Opensearch deals with search and indexing well and is available as managed.
The service will be performing multiple tasks that are intermittent, it made sence to use serverless design for the steps involved. (Lambda function, api gateway)
Container Architecture with cluster resource is used to host the applications which allows scalability
