---
title: Kubernetes vs Mesos vs Swarm
date: 2018-09-17
tags:
- container
- scheduling

categories:
- container technologies
description: "A Look at Major Container Orchestration Engines"
---

## Introduction
如果您正在阅读本文，您可能会问自己什么是容器编排引擎，它们解决了哪些问题，以及它们之间的区别。本文将尝试对Kubernetes，Docker Swarm和Apache Mesos进行high-level overview，以及它们的一些显着的相似点和不同点。

## Container Orchestration Engines

虽然定义各不相同，但Kubernetes，Docker和Swarm都属于一类DevOps基础架构管理工具，称为Container Orchestration Engines（COE）。 COE在资源池和在这些资源上运行的应用程序容器之间提供抽象层。

与容器一起，COE解决的主要问题是如何在云或数据中心中采用多个离散资源，并将它们组合到一个池中，可以在其上部署各种应用程序。这些应用程序的范围可以从简单的三层Web体系结构到大规模数据摄取和处理，以及介于两者之间的所有内容。
这些工具中的每一个都提供不同的功能集，并且在成熟度，学习曲线和易用性方面各不相同。他们可以共享的一些高级功能包括：
- Container scheduling --- 其中包括执行启动和停止容器等功能;在集合资源中分配容器;收回失败的容器;将容器从故障主机重新平衡到健康主机，并通过容器扩展应用程序，无论是手动还是自动。
- High availability --- 应用程序和容器，或编排系统本身的高可用性。
- Health checks --- 确定容器或应用程序运行的健康性检测
- Service discovery --- 其用于确定分布式计算架构中的各种服务在网络上的位置。
- Load Balancing requests --- 无论是在集群内部生成还是从外部客户端生成的请求的负载均衡
- 将各种类型（网络，本地）存储附加到群集中的容器。

NOTE，此列表并非详尽无遗，而是表示编排引擎提供的某些高级服务。值得一提的是，虽然这里讨论的每个工具都会在某种程度上执行这些功能，但实现方式可能会有很大差异。

## Kubernetes Container Orchestration Capabilities
Kubernetes（也称为“k8s”）于2014年6月首次发布，用Go编写。从古希腊语翻译，Kubernetes这个词的意思是“舵手。”该项目起源于谷歌开源，并且基于他们大规模运行容器的经验。

在功能方面，它可能是本文中检查的三个选项中最本地集成的。 Kubernetes使用非常广泛，背后有一个庞大的社区。 Google将Kubernetes用于自己的Container as a Service（CaaS）产品，称为Google Container Engine（GKE）。还有许多其他平台支持Kubernetes，包括Red Hat OpenShift和Microsoft Azure。

Docker是目前由Kubernetes支持的最普遍的容器引擎，但也支持CoreOS rkt（发音为“rocket”）。

Kubernetes使用基于YAML的部署模型。除了在主机上调度容器之外，Kubernetes还提供许多其他功能。

主要功能包括内置自动扩展，负载平衡，卷管理和秘密管理。此外，还有一个Web UI可帮助管理和排除群集故障。通过包含这些功能，Kubernetes通常比Swarm或Mesos需要更少的第三方软件。

将Kubernetes与Swarm和Mesos区分开来的还是“pods”的概念，它是一组容器，它们被组合在一起构成Kubernetes术语中的“服务”。

虽然可以将Kubernetes主服务器配置为高可用性集群，但这不像单节点主服务器那样受支持，并且是Kubernetes的高级用例。

Kubernetes的学习曲线有些陡峭，并且可以比Swarm更加努力地配置。部分由于其功能更紧密的集成，Kubernetes有时被认为比这里讨论的其他两个编排引擎更“opinionated”。

## Swarm Container Orchestration Capabilities
Docker Swarm是Docker的本机Container Orchestration Engine。最初于2015年11月发布，它也是用Go编写的。 Swarmkit是版本1.12中包含的Swarm的Docker本机版本，对于那些希望使用Swarm的人来说，这是Docker的推荐版本。

Swarm与Docker API紧密集成，非常适合与Docker一起使用。适用于单个主机泊坞窗群集的相同原语与Swarm一起使用。这可以简化容器基础架构的管理，因为不需要配置单独的编排引擎，也不需要重新学习Docker概念才能使用Swarm。

与Kubernetes一样，Swarm有一个使用Docker Compose的基于YAML的部署模型。其他值得注意的功能包括群集自动修复，使用DNS覆盖网络，通过使用多个主服务器实现高可用性，以及使用带有证书颁发机构的TLS进行网络安全。

在撰写本文时，Swarm尚不支持本机自动缩放或外部负载平衡。扩展必须手动完成或通过第三方解决方案完成。同样，Swarm包括入口负载平衡，但外部负载平衡将通过使用第三方负载平衡器（如AWS ELB）完成。另外值得注意的是缺少Swarm的Web界面。

## Mesos Container Orchestration Capabilities
Apache Mesos 1.0版本于2016年7月发布，但它的历史可以追溯到2009年，当时它最初是由加州大学伯克利分校的博士生开发的。与Swarm和Kubernetes不同，Mesos是用C ++编写的。

Mesos与前面提到的前两个有些不同，因为它需要更多的分布式方法来管理数据中心和云资源。 Mesos可以拥有多个主服务器，这些主服务器使用Zookeeper来跟踪主服务器中的集群状态，并形成高可用性集群。

其他容器管理框架可以在Mesos上运行，包括Kubernetes，Apache Aurora，Chronos和Mesosphere Marathon。此外，Mesosphere DC / OS是一个分布式数据中心操作系统，基于Apache Mesos。

这意味着Mesos采用更加模块化的方法来管理容器，允许用户在应用程序类型和运行规模方面具有更大的灵活性。

Mesos可以扩展到数万个节点，并被Twitter，Airbnb，Yelp和eBay等用户使用。 Apple甚至拥有自己的基于Mesos的专有框架，名为Jarvis，用于驱动Siri。

Mesos中可用的一些值得一提的功能是支持多种类型的容器引擎，包括Docker及其自己的“Containerizer”，以及Web UI，以及在多个操作系统上运行的能力，包括Linux，OS X，甚至是Windows。

由于其复杂性和灵活性，Mesos拥有比Docker Swarm更陡峭的学习曲线。但是，同样的灵活性和复杂性也是允许像Twitter和Airbnb这样的公司使用Mesos来管理其大规模应用程序的优势。

## Conclusion
如果您只是想要使用编排引擎启动并运行并测试，那么Doc​​ker Swarm可能是一个不错的选择。当你准备深入研究这个主题，或者可能部署一些倾向于工业级的东西时，请看看Kubernetes。如果灵活性和大规模是您的目标，那么请考虑Apache Mesos。

本文翻译[这篇文章](https://www.sumologic.com/devops/kubernetes-vs-mesos-vs-swarm/), 如有侵权请告知。


When you’re ready to learn a bit more, take a look at Comparison of Container Schedulers, and [Kubernetes, Mesos, and Swarm: Comparing the Rancher Orchestration Engine Options.](http://rancher.com/comparing-rancher-orchestration-engine-options/)
