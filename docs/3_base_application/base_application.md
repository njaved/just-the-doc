---
title: AWS Infrastructure
layout: page
nav_order: 4
nav_enabled: true
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# AWS Infrastructure 
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
5. Customize terraform.tfvars configuration file. Before editing terraform.tfvars, collect the following information in the Table-1 below from your existing environment


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


#### Table-1 (terraform.tfvars)

6. Edit file `terraform.tfvars` and fill the variables from the *Table-1*.

7. Customize `terraform.tf` configuration file. Before editing `terraform.tf`:

    a. You will need a bucket to hold terraform working assets. Decide on the name, e.g. `cdc-nbs-terraform-<EXAMPLE_ACCOUNT_NUM>`

    b. Create an S3 bucket e.g. `cdc-nbs-terraform-<EXAMPLE_ACCOUNT_NUM>` on the AWS console

    c. Edit file `terraform.tf`, change the bucket name and `SITE_NAME` in the file as suggested in the *Table-2*


#### Table-2

| Parameter | Template Value                                               | Example                                            | Description                                                                 |
|-----------|--------------------------------------------------------------|----------------------------------------------------|-----------------------------------------------------------------------------|
| bucket    | `cdc-nbs-terraform-<EXAMPLE_ACCOUNT_NUM>`                    | `cdc-nbs-terraform-12345678901213`                | S3 bucket to store infrastructure artifacts                                 |
| key       | `cdc-nbs-SITE_NAME-modern/infrastructure-artifacts`          | `cdc-nbs-fts3-modern/infrastructure-artifacts`    | Path for the artifacts inside the S3 bucket. The bucket needs to exist before running `terraform apply`, but the path will be created automatically. |


8. Review the inbound rules on the security groups attached to your database instance and ensure that the CIDR you intend to use with your NBS 7 VPC (modern-cidr) is allowed to access the database.

    a. For e.g if the modern-cidr from the Table-1 is 10.20.0.0/16, there should be at least one rule in a security group associated to your database that allows MSSQL inbound access from your modern-cidr block


9. Make sure you are authenticated to AWS. Confirm access to the intended account using the following command. (More information about authenticating to AWS can be found at the following link.)
```
$ aws sts get-caller-identity
{
    "UserId": "AIDBZMOZ03E7R88J3DBTZ",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/lincolna"
}
```

10. Terraform stores its state in an S3 bucket. The commands below assume that you are running Terraform authenticated to the same AWS account that contains your existing NBS 6 application. Please adjust accordingly if this does not match your setup.

    a.Change directory to the account configuration directory if not already, i.e. the one containing terraform.tfvars, and terraform.tf

    ```
    cd terraform/aws/nbs7-mySTLT-test
    ```

    b.Initialize Terraform by running:

    ```
    terraform init
    ```

    c. Run “terraform plan” to enable it to calculate the set of changes that need to be applied:

    ```
    terraform plan
    ```

    Note: One warning is expected (due to a bug in the Hashicorp EKS module):

    ```
    Warning: Argument is deprecated
    with module.eks_nbs.module.eks.aws_eks_addon.this["aws-ebs-csi-driver"],
    on .terraform/modules/eks_nbs.eks/main.tf line 392, in resource "aws_eks_addon" "this":
    392:   resolve_conflicts        = try(each.value.resolve_conflicts, "OVERWRITE")
    The "resolve_conflicts" attribute can't be set to "PRESERVE" on initial resource creation. Use "resolve_conflicts_on_create" and/or "resolve_conflicts_on_update" instead
    ```

    d. Review the changes carefully to make sure that they 1) match your intention, and 2) do not unintentionally disturb other configuration on which you depend. Then run “terraform apply”:

    ```
    terraform apply
    ```

    e. If terraform apply generates errors, review and resolve the errors, and then rerun step d.

11. Verify Terraform was applied as expected by examining the logs

12. Verify the newly created VPC and subnets were created as expected and confirm that the CIDR blocks you defined exist in the Route Tables

13. Verify the EKS Kubernetes cluster was created by selecting the cluster and inspecting Resources->Pods, Compute (expect 30+ pods at this point, and 3-5 compute nodes as per the min/max nodes defined in terraform/aws/app-infrastructure/eks-nbs/variables.tf)

14. Now that the infrastructure has been created using terraform, deploy Kubernetes (K8s) support services in the Kubernetes cluster via the following steps

    a. Start the Terminal/command line:

        - Make sure you are still authenticated with AWS (reference the following Configuration and credential file settings).

        - Authenticate into the Kubernetes cluster(EKS) using the following command and the cluster name you deployed in the environment


        ```
        aws eks --region us-east-1 update-kubeconfig --name <clustername> # e.g. cdc-nbs-sandbox
        ```
        
        Note: You should see a line “Added new context ….“.

        - If the above command errors out, check

            1. there are no issues with the AWS CLI installation

            2. you have set the correct AWS environment variables

            3. you are using the correct cluster name (as per the EKS management console)

        b. Run the following command to check if you are able to run commands to interact with the Kubernetes objects and the cluster.

        ```
        kubectl get pods --namespace=cert-manager
        ```

        The above command should return 3 pods. If it doesn’t refresh the AWS credentials and repeat steps in 14 a.

        ```
        kubectl get nodes
        ```
        The above command should list 3 worker nodes for the cluster.

15. Congratulations! You have installed your core infrastructure and Kubernetes cluster! Next, we will configure your cluster using helm charts.