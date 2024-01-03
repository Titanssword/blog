---
title: Dynamic Resource Allocation algorithms for container-based service computing
date: 2018-07-10
tags:
- container
- resource allocation 

categories:
- container technologies
description: "Cloud computing and virtualization technologies play important roles in modern service-oriented computing paradigm. More conventional services are being migrated to virtualized computing environments to achieve flexible deployment and high availability."
---
## Introduction
Nowadays, the on-demand virtualized resources are offered without any delay, according to actual requirements in real-time. To make the cloud platform more flexible and cost efficient. A few management frameworks are proposed to enable cluster resource sharing across workloads. Most are suitable in building Infrastructure-as-a-servcie(Iaas) cloud service model. To make the cloud platform more flexible and cost efficent, virtual resources may be added or removed from the cloud platform at any time. Therefore, statically setting runtime parameters often leads to an unbalanced resources. Conversely, Dynamic provisioning of resources on a fine-grained basis has significant efficiency and cost adventages. Usually, a cluster management module is deployed to allocate and assgin tasks to maximize the utilization of available resources. It allocates the proper amount of resources for each workloads, and select specific servers that will satisfy a given allocation.

## About container
containers allow servcies to run in isolated environments without the extra overhead of running entirely separate operating systems. Unfortunately, the problems of how to effectively management computing resource for containers remain open, bacause multiple applications sharing the same resources can result in substantial resource contention among the applications in the containers.

In a typical production environment, large group of remote servers in a data center work under complex hierarchical layers with cross-domain cooperation. Therefore, effective job scheduling, load balancing and resource allocation become critical to ensure the scalability and high availability of a virtualization computing environment.

In a CaaS framework, a container is the basic component that consitutes the business workflow, while a physical node (Usually a server) is the fundamental carrier to deploy and execute containers.

## What they do in this paper

They proposed a node selection algorithm for container deployment(NSCD), where a Fuzzy Inference System(FIS) is applied to dynamically predict the most proper node (server) where the selected containers will be deployed.
