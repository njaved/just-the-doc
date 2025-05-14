---
title: Quickstart
layout: page
nav_order: 1
nav_enabled: true
---

This quick guide provides a simple step-by-step approach for deploying the NBS 7.9 infrastructure and microservices in an AWS environment. It is intended for experienced administrators who are familiar with AWS, Kubernetes, Helm, and Terraform. 

This guide is NOT meant for a production deployment. Please review System Administrator Guide for a full production deployment and guidelines.


Prerequisites
Tools to Install
AWS CLI (v2.15+)

Terraform (v1.5.5)

Helm (v3.12+)

kubectl (v1.27+)

eksctl (optional but recommended)

Environment Requirements
AWS Account with NBS 6.0.15 access (or newer)

DNS routing infrastructure

IAM Roles for Terraform and Kubernetes

Access to NBS 6 databases (SQL Server)

S3 bucket for Terraform state

1. Set Up AWS Infrastructure - Terraform
Deployment
NBS7 Infrastructure in AWS
Prepare the Directory


mkdir -p ~/nbs-setup/terraform/aws/nbs7-mySTLT-test
cd ~/nbs-setup/terraform/aws/nbs7-mySTLT-test
Download Terraform Configuration
Clone the infrastructure repo:



git clone https://github.com/CDCgov/NEDSS-Infrastructure.git
Copy standard template:



cp -pr terraform/aws/samples/NBS7_standard terraform/aws/nbs7-mySTLT-test
Customize Variables
Edit terraform.tfvars with your environment-specific values (account ID, VPC CIDRs, DNS, IAM roles).

Initialize and Apply Terraform


terraform init
terraform plan
terraform apply
Validate Infrastructure
Confirm VPC, EKS cluster, subnets, and node groups are created.

Run:



kubectl get nodes
2. Deploy Core Kubernetes Services - Helm
Install NGINX Ingress


helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx
Install Cert Manager (Optional)


helm repo add jetstack https://charts.jetstack.io
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true
Install Cluster Autoscaler (Optional)


helm repo add autoscaler https://kubernetes.github.io/autoscaler
helm install cluster-autoscaler autoscaler/cluster-autoscaler -n kube-system
Verify Services are Running


kubectl get pods -A
3. Deploy NBS Microservices - Helm
Deploy Elasticsearch


helm install elasticsearch -f ./elasticsearch-efs/values.yaml elasticsearch-efs
Deploy Page-Builder API


helm install page-builder-api -f ./page-builder-api/values.yaml page-builder-api
Deploy Modernization API


helm install modernization-api -f ./modernization-api/values.yaml modernization-api
Deploy NiFi


helm install nifi -f ./nifi-efs/values.yaml nifi-efs
Deploy NBS Gateway


helm install nbs-gateway -f ./nbs-gateway/values.yaml nbs-gateway
Verify Services
Confirm all pods are running before moving on.



kubectl get pods -A
4. Validate Installation
Manual Tests
Login to the NBS UI (e.g., https://app.example.com/nbs/login)

Check basic patient search functionality.

Automated Tests
Use nbs-test-api.sh and nbs-test-webui.sh for basic API and UI smoke tests.

5. Go Live and Monitor
We recommend before going live to review the full detailed installation guide

Verify DNS routing and load balancer configurations.

Monitor initial traffic, logs, and database connectivity.

6. Cleanup and Maintenance
Regularly validate cluster health.

Ensure data retention and backup policies are in place.

Support
For support, contact NBSSupport@cdc.gov.

For ongoing updates, check the GitHub repo for new releases.