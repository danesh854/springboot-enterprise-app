# Enterprise Spring Boot DevOps Platform

## Project Overview

This project demonstrates a production-grade DevOps implementation for deploying a Spring Boot microservice application on AWS EKS using modern CI/CD, Infrastructure as Code, security scanning, monitoring, and zero downtime deployment practices.


## Technology Stack

### Cloud
- AWS

### Infrastructure as Code
- Terraform
- S3 Remote Backend
- DynamoDB State Locking

### CI/CD
- Jenkins Pipeline
- GitHub Webhook

### Build
- Maven

### Code Quality
- SonarQube

### Security
- Trivy Vulnerability Scanner

### Containerization
- Docker
- DockerHub Registry

### Orchestration
- Kubernetes
- AWS EKS

### Deployment Strategy
- Blue Green Deployment

### Monitoring
- Prometheus
- Grafana


## Architecture Flow


Developer
    |
    |
GitHub Repository
    |
    |
GitHub Webhook
    |
    |
Jenkins Pipeline
    |
    |---- Maven Build
    |
    |---- SonarQube Analysis
    |
    |---- Quality Gate
    |
    |---- Docker Build
    |
    |---- Trivy Scan
    |
    |---- Docker Push
    |
AWS EKS Cluster
    |
    |
ALB Ingress Controller
    |
    |
Kubernetes Service
    |
    |
Blue / Green Deployments


Prometheus ---> Grafana Monitoring


## CI/CD Pipeline Stages

1. Checkout source code
2. Maven package build
3. Static code analysis using SonarQube
4. Quality gate validation
5. Docker image creation
6. Vulnerability scanning using Trivy
7. DockerHub image push
8. Kubernetes Blue Green deployment
9. Deployment verification


## Kubernetes Features

- Namespace isolation
- Deployment management
- Service discovery
- ALB Ingress
- Horizontal Pod Autoscaling
- Rolling updates
- Blue Green releases


## Infrastructure

AWS Infrastructure provisioned using Terraform:

- VPC
- Public Subnets
- Private Subnets
- Internet Gateway
- NAT Gateway
- IAM Roles
- EKS Cluster
- Worker Nodes


## Terraform Backend

Terraform remote state is managed using:

- Amazon S3 bucket
- DynamoDB state locking


## Monitoring

Prometheus collects:

- Node metrics
- Pod metrics
- Kubernetes cluster metrics

Grafana provides visualization dashboards.


## Deployment Strategy

Blue Green deployment approach is used:

- Existing environment serves traffic
- New version deployed separately
- Health verification performed
- Traffic switched using Kubernetes service selector
- Instant rollback supported


## Author

Danesh Kabade
DevOps Engineer
