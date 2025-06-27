---
title: NND Service (Data Sync)
layout: page
parent: Microservices Deployment
nav_order: 8
nav_enabled: true
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Deploy Data Sync Service API via helm chart
This guide sets out the detailed steps to install NBS 7 Data Sync service that will be used to extract the data for NNDSS and STLT Reporting needs. The Data Sync service provides a secure API to connect to the databases in the NBS cloud, with an API endpoint service and without interrupting any operations On-Prem at Jurisdictions.

### Data Sync Microservice

1. Please use the values file supplied as part of [nbs-helm-<release>.zip](https://github.com/CDCgov/nbs-helm/releases). Use this [link](https://github.com/CDCgov/nbs-helm/releases) to download the zip file (scroll down to the Assets listed for the latest or previous releases). The `values.yaml` file should be under `charts\nnd-service\values.yaml`.  
   Values for *ECR repository, ECR image tag, db server endpoints, and ingress host* should be provided in the `values.yaml` file.

2. Confirm that the following DNS entry were created and pointed to the network load balancer in front of your Kubernetes cluster (make sure this is the ACTIVE NLB provisioned via nginx-ingress in the base install steps). This should be done in your authoritative DNS service (e.g., Route 53).  
   Please replace [http://example.com](http://example.com) with the appropriate domain name in the `values.yaml` file.  
   NND service Application – e.g. `nndservice.example.com`

3. Update the image repository and tag with the following:

   ```yaml
   image:
     repository: "quay.io/us-cdcgov/cdc-nbs-modernization/nnd-data-exchange-service"
     pullPolicy: IfNotPresent
     tag: <release-version-tag> e.g v1.0.1
   ```

4. 
