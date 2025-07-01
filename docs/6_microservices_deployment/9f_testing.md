---
title: API Testing
layout: page
parent: Case Notification
nav_order: 6
nav_enabled: true
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

### Ingress setup

- All 3 case notification services are using the same data ingestion ingess. Use this existing setup as needed
  - **Data Extraction**: [NEDSS-Helm/charts/dataingestion-service/templates/ingress.yaml at 10623c0d9788a6513bd51f4b6ed4eb0f79b30a2f · CDCgov/NEDSS-Helm](https://github.com/CDCgov/NEDSS-Helm/blob/10623c0d9788a6513bd51f4b6ed4eb0f79b30a2f/charts/dataingestion-service/templates/ingress.yaml)

  - **Case Notification**: [NEDSS-Helm/charts/dataingestion-service/templates/ingress.yaml at 10623c0d9788a6513bd51f4b6ed4eb0f79b30a2f · CDCgov/NEDSS-Helm](https://github.com/CDCgov/NEDSS-Helm/blob/10623c0d9788a6513bd51f4b6ed4eb0f79b30a2f/charts/dataingestion-service/templates/ingress.yaml)

  - **HL7 Parser**: [NEDSS-Helm/charts/dataingestion-service/templates/ingress.yaml at 10623c0d9788a6513bd51f4b6ed4eb0f79b30a2f · CDCgov/NEDSS-Helm](https://github.com/CDCgov/NEDSS-Helm/blob/10623c0d9788a6513bd51f4b6ed4eb0f79b30a2f/charts/dataingestion-service/templates/ingress.yaml)

### PHIMNS Properties

- For the services to be fully functional, the STLT need to provide CDC their PHIMS properties. This ensures the data in the `TransportQ_Out` table gets updated correctly when the data is being processed by the CDC Case Notification. These data can be pulled from the existing NND Rhapsody route’s Variable Manager on the STLT level.

  - **Require**
    - `nbs_certificate_url` – STLT most likely will have this hardcoded in the Rhapsody Variable Manager

  - **Optional** – these are optional, some STLT might have or need them:
    - `phin_encryption`
    - `phin_route`
    - `phin_signature`
    - `phin_public_key_address`
    - `phin_public_key_base_dn`
    - `phin_public_key_dn`
    - `phin_recipient`
    - `phin_priority`

- Once the data has been provided by the STLT, CDC should update the existing config on the CDC Case Notification config table:

  - **Table Name:** `NBS_Case_Notification_Config`
  - **Update record where config name is:** `NON_STD_CASE_NOTIFICATION`
  - **Update value:** Any values provided by STLT should be matched with the reflected column in the config table

  - **Update Script - DB:** `NBS_MSGOUTE`

    ```sql
    UPDATE NBS_Case_Notification_Config
    SET
      nbs_certificate_url    = 'REPLACE WITH VALUE, REMOVE IF NON IS PROVIDED',
      phin_encryption        = 'REPLACE WITH VALUE, REMOVE IF NON IS PROVIDED',
      phin_route             = 'REPLACE WITH VALUE, REMOVE IF NON IS PROVIDED',
      phin_signature         = 'REPLACE WITH VALUE, REMOVE IF NON IS PROVIDED',
      phin_public_key_address= 'REPLACE WITH VALUE, REMOVE IF NON IS PROVIDED',
      phin_public_key_base_dn= 'REPLACE WITH VALUE, REMOVE IF NON IS PROVIDED',
      phin_public_key_dn     = 'REPLACE WITH VALUE, REMOVE IF NON IS PROVIDED',
      phin_recipient         = 'REPLACE WITH VALUE, REMOVE IF NON IS PROVIDED',
      phin_priority          = 'REPLACE WITH VALUE, REMOVE IF NON IS PROVIDED'
    WHERE config_name = 'NON_STD_CASE_NOTIFICATION'
    ```

 
