---
title: NBS 7 Architecture and Microservices
layout: page
parent: Introduction
nav_order: 1
---

# NBS 7 Architecture and Microservices
The deployment of Modern NBS 7 will complement and build upon the existing NBS 6.0.15 (or newer version) system, integrating them seamlessly through the strangler fig pattern. Users will experience a smooth transition between the modern NBS features and legacy NBS.

The Modern NBS will be hosted on a separate Virtual Private Cloud (VPC) to prevent any disruptions to the existing Classic NBS. These two VPCs will be interconnected to facilitate RDS access and other communication requirements.

The architecture diagram below illustrates the key components of the Modernized NBS.