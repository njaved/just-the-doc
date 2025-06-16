---
title: Linkerd
layout: page
parent: AWS Infrastructure
nav_order: 4
nav_enabled: true
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Linkerd - Annotate default namespace
- a. Verify that linkerd is installed as part of the terraform infrastructure deployment by checking the pods status under the “linkderd” namespace. This will enable linkerd/MTLS on all the microservices installed in the follow steps
  ```bash
  kubectl annotate namespace default "linkerd.io/inject=enabled"
  ```
- b. Verify annotation is in place:
  ```bash
  kubectl get namespace default -o=jsonpath='{.metadata.annotations}'
  ```
  Should show “{"linkerd.io/inject":"enabled"}”
- c. If this is an update of the environment rather than a new install, restart the application pods in the default namespace for the linkerd sidecars to be injected into each pods. Should show 2/2 on the restarted pod.
