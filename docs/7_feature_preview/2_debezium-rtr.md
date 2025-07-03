---
title: Debezium
layout: page
parent: Real Time Reporting (Feature Preview)
nav_order: 2
nav_enabled: true
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Debezium Source Connector/debezium-connect
1. The helm chart for debezium-connect should be available under charts/debezium.
2. Validate image repository and tag:
   ```yaml
   image:
     repository: quay.io/debezium/connect
     tag: <release-version-tag> e.g v1.0.1
   ```
3. Configurations for the following should be on hand to update the values.yaml file- NBS_ODSE hostname, username, password and kafka bootstrap server names. Replace all "EXAMPLE_MSK_KAFKA_ENDPOINT" with Kafka endpoints, Replace all "nbs-db.private-EXAMPLE_DOMAIN" with SQL server domain name, Replace all instances of "EXAMPLE_DB_USER" with SQL Server user name, Replace all instances of "EXAMPLE_DB_USER_PASSWORD" with SQL Server password
   ```sql
   properties:
     bootstrap_server: "EXAMPLE_MSK_KAFKA_ENDPOINT"
    
   sqlserverconnector: 
     config: 
       database.hostname: "nbs-db.private-EXAMPLE_DOMAIN",
       database.port: "1433",
       database.user: "EXAMPLE_DB_USER",
       database.password: "EXAMPLE_DB_USER_PASSWORD",
       database.dbname: "nbs_odse",
       database.names: "nbs_odse",
       database.server.name: "odse",
       database.history.kafka.bootstrap.servers: "EXAMPLE_MSK_KAFKA_ENDPOINT",
       schema.history.internal.kafka.bootstrap.servers: "EXAMPLE_MSK_KAFKA_ENDPOINT"   
   env:
     - name: BOOTSTRAP_SERVERS
       value: "EXAMPLE_MSK_KAFKA_ENDPOINT"
   ```
5. Install pod
   ```bash
   helm install -f ./debezium/values.yaml debezium-connect ./debezium/
   ```
6. Verify if pod is running
   ```bash
   kubectl get pods
   ```
7. Validate service
  - a. This is an internal service with no ingress. Validation should be part of [NBS 7.6 Application Installation Guide  RTR Pipeline Validation](https://cdc-nbs.atlassian.net/wiki/spaces/NM/pages/edit-v2/951255219#RTR-Pipeline-Validation)
  - b. If the service has any trouble connecting with the database, run this command to reset the ConfigMap.
```bash
kubectl delete configmap cp-kafka-connect-sqlserver-connect
```
   
