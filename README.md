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

## 2. Provisioning the EKS Cluster
The EKS cluster was brought up using Terraform’s aws_eks_cluster and aws_eks_node_group resources.
Key settings included:
- Managed node groups with autoscaling
- Policies enabling EKS worker nodes to communicate with control plane
- Security groups for pod, node, and ALB communication

Once deployed, the kubeconfig was updated locally using:
aws eks update-kubeconfig --name <cluster-name> --region <region>

## 3. Deploying the Application
Both the backend and frontend components of the retail application were containerized and deployed to the EKS cluster.

Kubernetes objects created included:
- A dedicated namespace (retail-app)
- Deployment manifests for backend and frontend pods
- Service objects for internal communication
This ensured isolated and scalable application workloads.

## 4. ALB Ingress Setup
The AWS ALB Ingress Controller was installed to manage external traffic.
The ingress resource was configured to:
- Route HTTP (80) and HTTPS (443) requests
- Attach a certificate from AWS ACM
- Forward traffic to appropriate Kubernetes services
Example (simplified) ingress manifest:
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: retail-ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
spec:
  ingressClassName: alb
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: retail-frontend
                port:
                  number: 80

## 5. SSL Setup Using ACM
A public SSL certificate was issued from AWS Certificate Manager (ACM).
Steps included:
- Requesting a certificate for the retail domain
- Completing DNS validation via CNAME records in Namecheap
- Associating the certificate with the ALB
This enabled secure HTTPS access to the application.

## 6. DNS Handling with Namecheap
Instead of Route53, Namecheap managed DNS.

Records configured:
- An A record that pointed the domain name to the ALB’s DNS endpoint
- A CNAME record for certificate validation

Once DNS propagation completed, both HTTP and HTTPS traffic worked correctly.

## 7. CI/CD Pipeline Using GitHub Actions
A GitHub Actions pipeline automated Terraform operations.
The workflow:
- Pull Request triggers formatting and linting
- Terraform validation
- Terraform plan preview
- Approval stage
- Terraform apply to update the live infrastructure

This delivered a robust GitOps-style IaC workflow.

## 8. Troubleshooting and Resolutions
Key issues encountered and fixed:
- Incorrect ingress mapping → resolved by adjusting service ports
- ACM certificate not attaching → corrected ALB security group inbound rules
- Pods failing health checks → updated readiness & liveness probe configurations

## Conclusion

This deployment confirmed that:
- EKS can reliably host the retail application
- Infrastructure-as-code provides reproducibility and version control
- ALB integrates smoothly with Kubernetes ingress
- Namecheap DNS and ACM certificates work seamlessly
- CI/CD automation simplifies deployments
The system is now production-ready and can be extended further with enhanced monitoring, logging, and scaling.