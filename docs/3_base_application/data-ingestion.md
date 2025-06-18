---
title: Data Ingestion
layout: page
parent: Microservices Deployment
nav_order: 5
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
   ```bash
   IF NOT EXISTS(SELECT * FROM sys.databases WHERE name = 'NBS_DataIngest')
   BEGIN
       CREATE DATABASE NBS_DataIngest
   END
   GO
   USE NBS_DataIngest
   GO
   ```
2. Run the following script to create required permissions for nbs_ods user to NBS_DataIngest database:
   ```bash
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
