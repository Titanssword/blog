---
title: container scheduling
date: 2018-07-13
tags:
- container
- resource allocation
- scheduling

categories:
- container technologies
description: "This paper presents a study of ACO to implement a new scheduler for docker. The main contribution of this paper is an ACO-based algorithm, which distributes application containers over Docker hosts. It is to balancing the resource and finally leads to the better performance of applications."
---
## Introduction
Linux containers allow applications to run in complete isolation from one another without the extra overhead of running entirely separate operating systems. Linux容器允许应用程序彼此完全隔离运行，而无需运行完全独立的操作系统的额外开销。This approach eliminates memory overheads associated with virtualization and virtual machines and helps businesses run their day-to-day applications. software containers become the new pillars for software deployment and today Internet. Docker, a software container implemenation, has emerged not only as a new virtualization technologies but also an application delivery platform. 容器成为软件部署和今天互联网的新支柱。 Docker是一个软件容器实现，不仅是一种新的虚拟化技术，也是一种应用交付平台。 Container can be considered the virtualization at tht operating system level. Unlike virtual machine, containers are put into a kind of isolation which shares the host operating system's kernel. 下面是容器和虚拟机的比较。
![](https://github.com/Titanssword/Notes/blob/master/pic/containers/container%20vs%20vm.PNG?raw=true)

###CONTAINERS
Containers are an abstraction at the app layer that packages code and dependencies together. Multiple containers can run on the same machine and share the OS kernel with other containers, each running as isolated processes in user space. Containers take up less space than VMs (container images are typically tens of MBs in size), and start almost instantly.容器是应用层的抽象，它将代码和依赖关系打包在一起。多个容器可以在同一台机器上运行，并与其他容器共享操作系统内核，每个容器在用户空间中作为独立进程运行。容器占用的空间比虚拟机少（容器映像的大小通常为几十MB），并且几乎立即启动。
###VIRTUAL MACHINES
Virtual machines (VMs) are an abstraction of physical hardware turning one server into many servers. The hypervisor allows multiple VMs to run on a single machine. Each VM includes a full copy of an operating system, one or more apps, necessary binaries and libraries - taking up tens of GBs. VMs can also be slow to boot.
虚拟机（VM）是物理硬件的抽象，将一台服务器转变为多台服务器。虚拟机管理程序允许多台虚拟机在一台计算机上运行。每个VM都包含操作系统的完整副本，一个或多个应用程序，必要的二进制文件和库 - 占用数十GB。虚拟机也可能很慢启动。

Unfortunately, multiple applications sharing the same resources can result in substantial resource contention. 遗憾的是，共享相同资源的多个应用程序可能导致大量资源争用。 One way to mitigate this loss in performance is by ensuring quality of service(QoS) guaranteeing that the application of interest meets the performance requiremnets. 减轻性能损失的一种方法是确保服务质量（QoS），保证感兴趣的应用程序满足性能要求。

## Related work
scheduling algorithms for clusters may have different purposes. For example, some algorithms have designed to utilize the cluster's resources efficently. While others, have been designed to maximize the application's performance.

First, the monolithic algorithm is a scheduler with the single system to handle all task placement request. ->>>>Docker Swarm and SwarmKit.
Second, two-level scheduler. It allows the cluster to be divided into many sub-clusters. Each sub cluster may have its own scheduler. ->>>>> Apache Mesos
Third, a share-state scheduler is allowed to access to the cluster and let the scheduler to complete together in the form of free-for-all. It is fully distributed scheduler and there is no centralized resource allocation nor the central policy enforcement mechanism. ->>>> Google Omega

## Paper 1. Improvenment of Container scheduling for docker using Ant Colony Optimization
> This paper presents a study of ACO to implement a new scheduler for docker. The main contribution of this paper is an ACO-based algorithm, which distributes application containers over Docker hosts. It is to balancing the resource and finally leads to the better performance of applications.

ACO is a meta-heuristic algorithm which is widely adopted for Optimization and combinatory problems. But there is no study of using ACO to implement schedulers for a software container system yet, especially from the perspective of application workload deployment. ACO是一种元启发式算法，广泛应用于优化和组合问题。但是还没有研究使用ACO来实现软件容器系统的调度程序，特别是从应用程序工作负载部署的角度来看。

The main contribution of this paper is an ACO-based algorithm which spread application containers over Docker hosts to better balance the overall resource usages and Therefore lead to the better performance of applications compared to the current *greedy* scheduler.

#### Objectives
The scheduler tries to put task onto the available resources. So it always reduce the available resources every time it takes an action.  

优化目标只有一个， 资源利用率， CPU和内存， 已占用比上未占用的。

## Paper2. Container Oriented Job scheduling Using Linear programming Model
> This paper, **the container host energy conservation**, **the container image pulling costs from the image registry to the container image pulling costs from the image registry to the container hosts** and **the workload network transition costs from the clients to the container hosts** are evaluated in combination.


The key issue in the whole workflow in the designing of a scheduler to dispatch the workloads to container hosts, to meet the following criterions:

1. All the workloads from clients are containerized and processed in the hosts.
2. The workloads scheduled to a specific host cannot exceed the host's computing capacity.

The correlations between the server utilization and the electric power consumption have been widely investigated.
 这里 文章通过计算 利用率以及使用相关公式来 找到 能耗的 公式。

`min f(x, y) = f1(x) + \alpha f2(x) + \beta f3(x)`

### Paper3.  Genetic Algorithm for multi-objective Optimization of container allocation in cloud architecture
> In this paper, they proposed a genetic algorithm approach, using the Non-dominated Sorting Genetic Algorithm-II(NSGA-II), to optimize container allocation and elasticity management, motivated by the good results obtained with this algorithm in other resource management optimization problems in cloud architectures.

该文先介绍了 微服务这个架构 ，以及其好处。
Microservice pattern defines an application as a set of independent small and modular services each executing a single task, and the specification of the interoperability of these microservices to achieve the application requirements.
####  Benefits:
Making easier to ship and update applications, allowing independent updating and redeployment of parts of the application, facilitating closer development and operation teams,
Allowing continuous release cycles, simplifying the orchestration of applications across heterogeneous cloud data centers, and so on[open issues in scheduling microservices in the cloud

#### objectives:

1. The provisioning of new applications and the elasticity of currently deployed ones by maintaining a uniform distribution of the workload across the cluster. 通过在整个集群中保持工作负载的统一分布来提供新应用程序和当前部署的应用程序的弹性。
2. The performance of the deployed applications and their assigned resources by considering a suitable scalability level of their microservices and a uniform distribution of the microservices workload across their containers.部署的应用程序及其分配的资源的性能，通过考虑其微服务的合适可伸缩性级别以及跨容器的微服务工作负载的统一分布。
3. The reliability of the microservices by considering a suitable scalability level and a correct distribution avoiding single points of failure.通过考虑合适的可扩展性级别和正确的分布避免单点故障，微服务的可靠性。
4. The network overhead of the commuications between microservices by placing the containers of related microservicesin physical machines with short network distance. 通过将相关微服务的容器放置在具有短网络距离的物理机器上，微服务之间的通信的网络开销。

#### System Model
They consider a set of applications A, following a development pattern based on mivroservices. 他们根据基于mivroservices的开发模式考虑一组应用程序A.
Each of these applications is charactezied by the number of user requests and the microservices stack.  
