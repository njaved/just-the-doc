---
title: NBS7 Base Application
layout: page
nav_order: 4
nav_enabled: true
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# NBS 7 Base Application  
### Installation Guide & Smoke Tests
{: .no_toc }

Installation of NBS 7 has three major phases. In the first phase you set up the environment and infrastructure, including k8s, in AWS, 
primarily by using Terraform. In the second phase you start up the workloads/pods in k8s, primarily using Helm (with a bit more Terraform).

Finally, you validate the release by performing a smoke test and inspecting monitoring/console/admin interfaces for the NBS service and
NiFi.


### NBS 7 Deployment
This guide sets out the detailed steps to installing NBS 7 end to end.
Make sure you’re using the latest documentation from NBS Central ( NBS Central)
1. System Administrator Guide (this document)
2. User Guide
3. Release Notes

Installation is divided into two sections:

1. Terraform Deployment - installs the core infrastructure
2. Deployment to Kubernetes - bootstrap core kubernetes infrastructure services & install and configure microservices (elasticSearch,
page-builder, modernization-api, nifi, nbs-gateway, data-ingestion, RTR services and nnd)

### Terraform Deployment 
1. Download the Terraform configuration package from GitHub. Make sure you go through the release page and see what's included in
the current release on CDCgov/NEDSS-Infrastructure
2. Open bash/mac/cloudshell/powershell and unzip the current version file downloaded in the previous step named nbs-infrastructure-
vX.Y.Z zip.
3. Create a new directory with an easily identifiable name e.g <nbs7-mySTLT-test> in /terraform/aws/ to hold your environment specific
configuration files
4. Copy terraform/aws/samples/NBS7_standard to the new directory and change into the new directory (Note: the samples directory
contains other options than “standard”, view the README file in that directory to chose most appropriate starting point)
```js
cp -pr terraform/aws/samples/NBS7_standard terraform/aws/nbs7-mySTLT-test
cd terraform/aws/nbs7-mySTLT-test
```


**NOTE**: Before editing **terraform.tfvars** and **terraform.tf** files below, you may reference detailed information for each TF module under the
“terraform/aws/app-infrastructure” in a readme file in each modules directory. Do not edit files in the individual modules.

### Terraform Variables Table

| **Parameter**                         | **Template Value**                            | **Example**                              | **Description**                                                                                                                                     |
|--------------------------------------|------------------------------------------------|------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| `target_account_id`                  | `EXAMPLE_ACCOUNT_ID`                          | `123456789012`                           | Account ID for the infrastructure deployment [AWS Account ID](https://us-east-1.console.aws.amazon.com/billing/home?region=us-east-1#/account)     |

#### Modern Config

| **Parameter**              | **Template Value**              | **Example**                       | **Description**                                                                                                                                     |
|---------------------------|----------------------------------|-----------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| `modern-cidr`             | `10.OCTET2a.0.0/16`             | `10.70.0.0/16`                    | 1. Assign a new CIDR range for the "modern" VPC using local addressing conventions with room for at least 4 x /24 subnets. Replace all references to `OCTET2a` with a CIDR from your organization if the `10.x.x.x` block is used, otherwise use a CIDR meeting subnet requirements. <br> 2. When the terraform is applied later, it will create a new VPC for the required resources. |
| `modern-private_subnets` | `["10.OCTET2a.1.0/24", "10.OCTET2a.3.0/24"]` | `10.70.1.0/24, 10.70.3.0/24`     | Assign a new modern private subnet CIDR range                                                                                                       |
| `modern-public_subnets`  | `["10.OCTET2a.2.0/24", "10.OCTET2a.4.0/24"]` | `10.70.2.0/24, 10.70.4.0/24`     | Assign a new modern public subnet CIDR range                                                                                                        |

#### Legacy Config

| **Parameter**                         | **Template Value**                 | **Example**                             | **Description**                                                                                                            |
|--------------------------------------|-----------------------------------|-----------------------------------------|----------------------------------------------------------------------------------------------------------------------------|
| `legacy-cidr`                        | `10.OCTET2b.0.0/16`               | `10.71.0.0/16`                          | Existing VPC CIDR for NBS classic application `legacy-cidr`                                                               |
| `legacy-vpc-id`                      | `vpc-LEGACY-EXAMPLE`             | `vpc-12345678901234567`                | Existing NBS Classic application VPC ID `legacy-vpc-id`                                                                   |
| `legacy_vpc_private_route_table_id` | `rtb-PRIVATE-EXAMPLE`            | `rtb-1234567890abcdef`                 | This is the route table used by the subnets to which the database is attached. (Assuming the RDS instance is on private subnets with corresponding route tables) |
| `legacy_vpc_public_route_table_id`  | `rtb-PUBLIC-EXAMPLE`             | `rtb-fedcba0987654321`                | This is the route table used by the subnets the application servers and/or load balancer are attached to (assume these are on “public” subnets)   |

#### Other Key Parameters

| **Parameter**              | **Template Value**                                  | **Example**                                 | **Description**                                                                                          |
|---------------------------|-----------------------------------------------------|---------------------------------------------|----------------------------------------------------------------------------------------------------------|
| `tags - Environment`      | `EXAMPLE`                                           | `fts3`                                      | Target environment                                                                                       |
| `aws_admin_role_name`     | `AWSReservedSSO_AWSAdministratorAccess_EXAMPLE_ROLE` | `AWSReservedSSO_AWSAdministratorAccess_12345678abcdefg` | This is the role your IAM/SSO user assumes when logged in. Get the value using: `aws sts get-caller-identity` |
| `fluentbit_bucket_prefix` | `EXAMPLE-fluentbit-bucket`                          | `fts3-fluentbit-bucket`                     | An S3 bucket that will be created to capture consolidated logs via FluentBit                            |
