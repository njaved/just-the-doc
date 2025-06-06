---
title: Quickstart
layout: page
nav_order: 1
nav_enabled: true
---

# Quickstart
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


This quick guide provides a simple step-by-step approach for deploying the NBS 7 infrastructure and microservices in an AWS environment. It is intended for experienced administrators who are familiar with AWS, Kubernetes, Helm, and Terraform. 

This guide is NOT meant for a production deployment. Please review [System Administrator Guide](https://cdc-nbs.atlassian.net/wiki/spaces/NM/pages/1592754177/NBS+7.10+System+Administrator+Guide) for a full production deployment and guidelines.

### Following resources will be installed and configured with this guide

#### Terraform
- Modern VPC, Subnets, Route Tables
- EKS Cluster, Nodes
- Network Load Balancer
- MSK
- Amazon Prometheus
- Amazon Grafana
- EFS
- KMS
- S3 Bucket

#### Manual
- Route53 Updates

#### NBS7 Core Services
1. Elasticsearch - For lightning-fast searches.
2. Modernization API - This service incorporates essential modern NBS features such as patient search, event search, patient profile, investigations, etc.
3. Nifi - Populates Elasticsearch indices from the NBS database.
4. NBS Gateway - Efficiently manages intricate strangler routing logic between modern and legacy NBS.
5. Data Ingestion - Enables NBS to seamlessly ingest HL7 data from labs and other entities into the NBS system.
6. Keycloak - Primary Identity Provider (IDP). Also used for token management and SSO integration, for example, OAuth, SAML integration with Okta, etc.

## Prerequisites
Tools to Install
- AWS CLI (v2.15+)
- Terraform (v1.5.5)
- Helm (v3.12+)
- kubectl (v1.27+)
- eksctl (optional but recommended)

## Environment Requirements
- AWS Account with NBS 6.0.15 access (or newer)
- DNS routing infrastructure
- IAM Roles for Terraform and Kubernetes
- Access to NBS 6 databases (SQL Server)
- S3 bucket for Terraform state

## Set Up AWS Infrastructure - Terraform


### Prepare the Directory
{: .no_toc }
```js
mkdir -p ~/nbs-setup/terraform/aws/nbs7-mySTLT-test
cd ~/nbs-setup/terraform/aws/nbs7-mySTLT-test
```

### Download Terraform Configuration
{: .no_toc }
Clone the infrastructure repo:
```js
git clone https://github.com/CDCgov/NEDSS-Infrastructure.git
```
Copy standard template:
```js
cp -pr terraform/aws/samples/NBS7_standard terraform/aws/nbs7-mySTLT-test
```

### Customize Variables
{: .no_toc }
- Update the terraform.tfvars and terraform.tf with your environment-specific values by following the instructions [here](https://github.com/CDCgov/NEDSS-Infrastructure/blob/main/terraform/aws/samples/NBS7_standard/README.md).

> ℹ️ **Review the inbound rules** on the security groups attached to your database instance and ensure that the CIDR you intend to use with your NBS 7 VPC (`modern-cidr`) is allowed to access the database.

### Initialize and Apply Terraform
{: .no_toc }
```js
terraform init
terraform plan
terraform apply
```

### Validate Infrastructure
{: .no_toc }
- Confirm VPC, EKS cluster, subnets, and node groups are created.
- Verify EKS cluster authentication and running pods & nodes:
```js
aws eks --region us-east-1 update-kubeconfig --name <clustername> # e.g. cdc-nbs-sandbox
kubectl get pods --namespace=cert-manager
kubectl get nodes
```

## Deploy Core Kubernetes Services - Helm
### Install NGINX Ingress
{: .no_toc }
```js
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx
kubectl --namespace ingress-nginx get services -o wide -w ingress-nginx-controller
kubectl get pods -n=ingress-nginx
```

### Create DNS Entries in Route53
{: .no_toc }
- Modernized NBS application pointed to the new network load balancer in front of your Kubernetes cluster
```js
app.<site_name>.<domain>.com
```
- Data Services pointed to the new network load balancer in front of your Kubernetes cluster
```js
data.<site_name>.<domain>.com
```

### Install Cert Manager (Optional)
{: .no_toc }
```js
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true
```

### Install and Verify LinkerD (Optional)
{: .no_toc }
```js
kubectl annotate namespace default "linkerd.io/inject=enabled"
kubectl get namespace default -o=jsonpath='{.metadata.annotations}'
```

### Install Cluster Autoscaler (Optional)
{: .no_toc }
```js
helm repo add autoscaler https://kubernetes.github.io/autoscaler
helm install cluster-autoscaler autoscaler/cluster-autoscaler -n kube-system
```

### Verify Services are Running
{: .no_toc }
```js
kubectl get pods -A
```

## Install Keycloak
**Create Keycloak Database. Make sure to update the database password.**
{: .no_toc }
```sql
use master
    IF NOT EXISTS(SELECT * FROM sys.databases WHERE name = 'keycloak')
    BEGIN
        CREATE DATABASE keycloak
    END
GO
    USE keycloak
GO

BEGIN
    CREATE LOGIN NBS_keycloak WITH PASSWORD = 'EXAMPLE_KCDB_PASS8675309';
    CREATE USER NBS_keycloak FOR LOGIN NBS_keycloak;
    EXEC sp_addrolemember N'db_owner', N'NBS_keycloak'
END

## Deploy NBS Microservices - Helm
### Deploy Elasticsearch
{: .no_toc }
```js
helm install elasticsearch -f ./elasticsearch-efs/values.yaml elasticsearch-efs
```

### Deploy Page-Builder API
{: .no_toc }
```js
helm install page-builder-api -f ./page-builder-api/values.yaml page-builder-api
```

### Deploy Modernization API
{: .no_toc }
```js
helm install modernization-api -f ./modernization-api/values.yaml modernization-api
```

### Deploy NiFi
{: .no_toc }
```js
helm install nifi -f ./nifi-efs/values.yaml nifi-efs
```

### Deploy NBS Gateway
{: .no_toc }
```js
helm install nbs-gateway -f ./nbs-gateway/values.yaml nbs-gateway
```

### Verify Services
{: .no_toc }
- Confirm all pods are running before moving on.
```js
kubectl get pods -A
```

## Validate Installation
### Manual Tests
{: .no_toc }
- Login to the NBS UI (e.g., https://app.example.com/nbs/login)
- Check basic patient search functionality.

### Automated Tests
{: .no_toc }
- Use nbs-test-api.sh and nbs-test-webui.sh for basic API and UI smoke tests.

## Go Live and Monitor
We recommend before going live to review the full detailed installation guide
- Verify DNS routing and load balancer configurations.
- Monitor initial traffic, logs, and database connectivity.

## Cleanup and Maintenance
- Regularly validate cluster health.
- Ensure data retention and backup policies are in place.

## Support
- For support, contact NBSSupport@cdc.gov.
- For ongoing updates, check the GitHub repo for new releases.
