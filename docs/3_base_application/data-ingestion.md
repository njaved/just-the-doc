---
title: Data Ingestion
layout: page
parent: Microservices Deployment
nav_order: 6
nav_enabled: true
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Data Ingestion Installation
This guide sets out the detailed steps to installing NBS 7 Data Ingestion, end to end.

### SQL Server - Databases
The DataIngestion service utilizes three databases: NBS_Msgoute, NBS_ODSE and NBS_DataIngest.
NBS_DataIngest is a new database essential for ingesting, validating Electronic Lab Reports (ELR), converting them into XML payloads, and integrating these XMLs into the NBS_MSGOUT database. It must be created before deploying the app on the EKS cluster.

### Manual Sql script for DI
Data Ingest DB creation and user permission in the following should be executed prior to the deployment of the data ingestion
1. create-nbs-dataingest-db.sql
   ```sql
   IF NOT EXISTS(SELECT * FROM sys.databases WHERE name = 'NBS_DataIngest')
   BEGIN
       CREATE DATABASE NBS_DataIngest
   END
   GO
   USE NBS_DataIngest
   GO
   ```
2. Run the following script to create required permissions for nbs_ods user to NBS_DataIngest database:
   ```sql
   USE [NBS_DataIngest]
   GO
   CREATE USER [nbs_ods] FOR LOGIN [nbs_ods]
   GO
   USE [NBS_DataIngest]
   GO
   ALTER USER [nbs_ods] WITH DEFAULT_SCHEMA=[dbo]
   GO
   USE [NBS_DataIngest]
   GO
   ALTER ROLE [db_owner] ADD MEMBER [nbs_ods]
   GO
   ```

### DI Liquibase:
- Data Ingestion comes with a built-in Liquibase that automatically handles any DB changes upon deployment.
  - DB changes detail can be reviewed here:
    - **DI Liquibase:** [NEDSS-DataIngestion/data-ingestion-service/src/main/resources/db at main · CDCgov/NEDSS-DataIngestion](https://github.com/CDCgov/NEDSS-DataIngestion/tree/main/data-ingestion-service/src/main/resources/db)
- Please refer to Deploy Data Ingestion via Helm chart for how to deploy DI.

### Liquibase DB change verification:
- To verify whether the database changes were applied, first ensure the DI container is stable and running; since the container manages Liquibase, it won't start if Liquibase fails.
- If there is failure by Liquibase, the DI pod will be unstable, and specific error can be found within the container log.

### Deploy Data Ingestion via Helm chart

1. Please use the values file supplied as part of [nbs-helm-<release>.zip](nbs-helm-<release>.zip) file. Use this [link](https://github.com/CDCgov/nbs-helm/releases) to download the zip file (scroll down to the **Assets** listed for the latest or previous releases). The `values.yaml` file should be under `charts\dataingestion-service\values.yaml`.  
   Values for **ECR repository**, **ECR image tag**, **db server endpoints**, **MSK(Kafka) bootstrap server**, and **ingress host** should be provided in the `values.yaml` file.
2. Confirm that the following DNS entry were created and pointed to the network load balancer in front of your Kubernetes cluster (make sure this is the **ACTIVE NLB** provisioned via `nginx-ingress` in the base install steps). This should be done in your authoritative DNS service (e.g., Route 53).  
   Please replace `http://example.com` with the appropriate domain name in the `values.yaml` file.  
   DataIngestion service Application – e.g., [dataingestion.example.com](http://dataingestion.example.com)
3. Update the image repository and tag with the following:
    ```yaml
    image:
      repository: "quay.io/us-cdcgov/cdc-nbs-modernization/data-ingestion-service"
      pullPolicy: IfNotPresent
      tag: <release-version-tag> e.g v1.0.1
    ```
4. To enable RTR ingress, make sure reporting service is true. Update it to false if RTR services are not used.
    ```yaml
    reportingService:
      enabled: "true"
    ```
5. Update the values file with the jdbc connection values in the following format. The database 'NBS_DataIngest' is a newly created database that is being used by the data ingestion service application. The databases ‘NBS_MSGOUTE’ and 'NBS_ODSE' are existing databases used for NBS batch processing. The dbserver value is just a database server endpoint. Please don't include the port number.
   ![data-ingestion-dbendpoint](/just-the-doc/docs/3_base_application/images/data-ingestion-dbendpoint.png)
   ```yaml
   jdbc:
      dbserver: "cdc-nbs-xxxxxxxxxxxx.us-east-1.rds.amazonaws.com"
      username: "EXAMPLE_DB_USER"
      password: "EXAMPLE_DB_USER_PASSWORD"
   ```
6. Use either one of the two Kafka broker endpoints ( Private endpoints - Plaintext) in the helm values file.
   ![data-ingestion-kafka-endpoint](/just-the-doc/docs/3_base_application/images/data-ingestion-kafka-endpoint.png)
   ```yaml
   kafka:
      cluster: "b-1.xxxxxxxxxxxxxx.amazonaws.com:9092"
   ```
7. Update the values.yaml to populate efsFileSystemId which is the EFS file system id from the AWS console. See image below.
   ![data-ingestion-efs](/just-the-doc/docs/3_base_application/images/data-ingestion-efs.png)
   ```yaml
   efsFileSystemId: "EXAMPLE_EFS_ID"
   ```
8. Keycloak Auth URI. Provide keycloak auth uri in the values.yaml file as shown below. In the default configuration this value should not need to change unless the name or namespace of the keycloak pod is modified.
   ```yaml
   authUri: "http://keycloak.default.svc.cluster.local/auth/realms/NBS"
   ```
9. SFTP Server for the manual file drop-off. It’s an OPTIONAL service. DI can poll ELRs from an external sFTP server. To configure this option, change sftp: enabled to "enabled" and put the appropriate host/username/password. If the SFTP server is unavailable or the manual drop-off option is not needed, the 'sftp: enabled' value should be 'disabled' or empty.
   ```yaml
   sftp:
     enabled: "enabled"
     host: "test.sftp-host.com"
     username: "testuser"
     password: "testpwd"
     elrFileExtns: "txt,hl7"
     filePaths: "/,/test-sub-folder"
   ```
For more information about sFTP support, please see: [data-ingestion-sftp-support](https://cdc-nbs.atlassian.net/wiki/spaces/NM/pages/1592755309)
10. Deploy DataIngestion helm chart:
After updating the values file, Run the following command to install dataingestion-service.
   ```bash
   helm install dataingestion-service -f ./dataingestion-service/values.yaml dataingestion-service
   ```
   - Note: Check to see if the pod for dataingestion-service is running using kubectl get pods
