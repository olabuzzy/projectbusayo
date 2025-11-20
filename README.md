# projectbusayo
Technical Report: Full Lifecycle Deployment of a Retail Platform on AWS EKS with ALB
http://a49440a8ac1a24b19af46f1618ba1ab4-893316149.eu-west-2.elb.amazonaws.com/

(If the URL does not open, the load balancer may have been stopped.)

## Overview

This document walks through the end-to-end workflow for launching a container-based retail application on Amazon Elastic Kubernetes Service (EKS) using an Infrastructure-as-Code (IaC) approach. It also describes how traffic routing was implemented using the AWS Application Load Balancer (ALB) Ingress Controller.

The major components of this project include:
- Provisioning the AWS infrastructure with Terraform
- Deploying a microservice-style retail application
- Configuring an ALB ingress controller
- Using ACM to issue SSL certificates
- Managing DNS with Namecheap
- Implementing GitHub Actions for CI/CD automation
- Providing user access via IAM roles

## 1. Setting Up Infrastructure with Terraform
Terraform was used to create all foundational AWS resources such as:
- VPC with subnets across multiple Availability Zones
- Public and private subnets
- Routing and internet gateway setup
- EKS cluster, node groups, and security configurations

Additional components included:
- Auto-scaling groups
- A DynamoDB table and S3 backend for Terraform state management
- IAM roles and permissions for Kubernetes and ALB
- ECR registry for application container images

Using Terraform ensured idempotent infrastructure builds with reliable rollback and recovery capability.