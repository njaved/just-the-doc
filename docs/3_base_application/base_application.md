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
