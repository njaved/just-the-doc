---
title: Organization Reporting
layout: page
parent: Real Time Reporting (Preview)
nav_order: 6
nav_enabled: true
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Organization-reporting-service
1. The helm chart for organization service should be available under charts/organization-reporting-service.
2. In the values.yaml, replace all occurrences of app.EXAMPLE_DOMAIN with the URL of your modern app as shown in [Table](/just-the-doc/docs/4_initial_kubernetes_deployment/1_nginx_ingress_deployment.html#deploy-nginx-ingress-controller-on-the-kubernetes-cluster).
3. Validate image repository and tag:
   ```yaml
   image:
     repository: "quay.io/us-cdcgov/cdc-nbs-modernization/data-reporting-service/organization-reporting-service"
     tag: <release-version-tag> e.g v1.0.1
   ```
4. Update jdbc, odse and kafka configurations
   ```yaml
   jdbc:
     username: "EXAMPLE_DB_USER"
     password: "EXAMPLE_DB_USER_PASSWORD"

   odse:
     dburl: "jdbc:sqlserver://<EXAMPLE_DB_ENDPOINT>:<PORT>;databaseName=NBS_ODSE;encrypt=true;trustServerCertificate=true;"
    
   rdb:
     dburl: "jdbc:sqlserver://<EXAMPLE_DB_ENDPOINT>:<PORT>;databaseName=rdb_modern;encrypt=true;trustServerCertificate=true;"
    
   kafka:
     cluster: "EXAMPLE_KAFKA_CLUSTER"
    
   featureFlag:
     elasticSearchEnable: '''true'''
   ```
5. Install pod
   ```bash
   helm install -f ./organization-reporting-service/values.yaml organization-reporting-service ./organization-reporting-service/
   ```
6. Verify if pod is running
   ```bash
   kubectl get pods
   ```
7. Validate service (on browser)
   ```
   https://data.<exampledomain>/reporting/organization-svc/status
   Expected: Organization Service Status OK
   ```
