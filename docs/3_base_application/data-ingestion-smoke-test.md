---
title: Data Ingestion Smoke Test
layout: page
parent: Data Ingestion
nav_order: 2
nav_enabled: true
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Data Ingestion Smoke Test

### Overview

The Data Ingestion service is integrated with the Keycloak server to authenticate the DI service's public API endpoints.  
As the **Client Credentials Grant type flow** is used for authentication in the DI Service, we require the **Client Id** and **Client Secret** values to create the JWT token and make API calls.

### Prerequisite

1. Keycloak server and the necessary configurations are setup.
2. Installation of Postman application to test the end to end flow.
   - If you already have postman application in your system, skip this step.
   - Visit the official Postman website at [www.postman.com](https://www.postman.com) and navigate to the **Downloads** section  
     [![Download Postman](https://www.postman.com/downloads/)](https://www.postman.com/downloads/) [Get Started for Free](https://www.postman.com/downloads/)
   - Click on the link to download the version appropriate for your OS. Once the download is complete open and install it accordingly.

### Scope

1. Only ELR Data that are HL7 messages with ORU RO1 Data type are in scope.
2. Only HL7 messages with versions 2.3.1 and 2.5.1 are in scope.
3. The DI service supports the transmit of HL7 messages with FHS header segments.

**Note:** Posting the same HL7 message more than once is allowed, but be aware that due to a duplicate check, the validation will fail within the Data Ingestion system.

### Run Data Ingestion Smoke Test

Open Postman application and click on import option. A pop up window shows up and then select `New-Data-Ingestion.postman_collection.json` file included in the release package and click on open to load Data Ingestion collection that has all API's related to the Data Ingestion service.

