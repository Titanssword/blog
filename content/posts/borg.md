---
title: Large-scale cluster management at Google with Borg
date: 2018-07-14
tags:
- container
- resource allocation
- scheduling

categories:
- container technologies
description: "Google's Borg system is a cluster manager that runs hundreds of jobs, from many thousands of different applications, across a number of clusters each with up to tens of thousands of machines.谷歌的Borg系统是一个集群管理器，可以运行数千个不同应用程序的数百个作业，这些作业分布在多个集群中，每个集群中有数万台计算机。It achieves high utilization by combining admission control, efficient task-packing, over-commitment, and machine sharing with process-level performance isolation."
---
## Introduction
The cluster management system we internally call **Borg admits**, **schedulers**, **starts**, **restarts**, and **monitors** the full range of applications that Google runs.

Borg provides three main benefits:
1. it hides the details of resource management and failure handing so its users can focus on application development instead
2. operates with very high reliability and availability and supports applications that do the same
3. lets us run workloads across tens of thousands of machines effectively.

Borg isolates users from most of these differences by determining where in a cell to run tasks, allocating their resources, installing their programs and other dependencies, monitoring their health, and restarting them if they fail.

## Parts
- Each job runs in *one Borg cell*, a set of machines that are managed as a unit. The machines in a cell belong to a single cluster, defined by the high-performance datacenter-scale network fabric that connects them.
- The workload -- Borg cells run a heterogenous workload with two main parts. The first is long-running services that should "never" go down, and handle short-lived latency-sersitive requests
  The second is batch jobs that take from a few seconds to a few days to complete, these are much less sensitive to short-term performance fluctuations.
- Prod -- the higher-priority Borg jobs as "production" (prod) ones, and the rest as "non-production" (non-prod). Most long-running server jobs are prod; most batch jobs are non-prod.
- Jobs and tasks -- A job's properties include its name, owner, and the number of tasks it has. Jobs can have contraints to force its tasks to run on machines with particular attributes such as processor architecture, OS version, or an external IP address. Each tasks maps to a set of Linux processes running in a container on a machine.
