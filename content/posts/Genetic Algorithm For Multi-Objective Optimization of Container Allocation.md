---
title: Genetic Algorithm for mulit-Objective Optimization of containers allocation in cloud architecture
date: 2018-07-21
tags:
- container
- resource allocation
- scheduling

categories:
- container technologies
description: "The use of containers in cloud architectures has become widespread, owing to advantages such as limited overheads, easier and faster deployment, and higher portability. Moreover, they present a suitable architectural solution for the deployment of applications created using a microservice development pattern. But the remain open issue is that ----- container resource allocation influences system performance and resource consumption, and so it is a key factor for cloud providers. In this paper, they propose a genetic algorithm approach, using the Non-dominated Sorting Genetic Algorithm 2"
---
## Introduction
Mainly talk about the Microservice architecture

The containerization of applications is one of the technologies enabling microservices architectures. microservices can be scaleb up by simply creating new containers until the desired by scalability level is achieved. Docker is one of the most successful implementations of container architectures.

Goals: ->>> consolidation	合并, 巩故, elasticity弹性, 弹力, load balancing, and scalability可扩展性.
Examples: ->>> Docker Swarm, Apache Mesos, and Google Kubernetes.
Four traditional research: ->>> System provisioning, system performance, reliability, and network overhead.

The nature of the problem: ->>> Resource management Optimization is an NP-complete problem, and must be addressed using meta-heuristic approaches. The optimal solution can be only obtained through an evaluation of all possible combinations. The literature reflects that evolutionary approaches, such as genetic algorithm(GA), are common solutions to resource management in the cloud. **Further more, the Non-dominated sorting genetic algorithm-2 is one of the most common solutiong for multi-Objective Optimization.**

They implement a container allocation strategy and automatic elasicity management(scalability level of the microservices) by the Optimization of four Objectives:
- The provisioning of new applications and the elasicity of currently deployed ones by maintaining a uniform distribution of the workload across the cluster.
- The performance of the deployed applications and their assigned resources by considering a suitable scalability level of their microservices and a uniform distribution of the microservices workload across their containers.
- The reliability of the microservices by considering a suitable scalability level and a correct distribution avoiding single points of failure.
- The network overhead of the communications between microservices by placing the containers of related microservices in physical machines with short network distance.

## System Model
