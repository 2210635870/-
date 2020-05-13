---
layout: post
title: "Kubernetes 核心技术概念"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### 1.  Kubernetes 对象

Kubernetes 对象是 Kubernetes 集群中的管理操作单元。Kubernetes 集群系统每支持一项新功能，引入一项新技术，一定会新引入对应的 Kubernetes 对象，支持对该功能的管理操作。例如副本集Replica Set 对应的 Kubernetes 对象是RS。

每个 Kubernetes 对象都有3大类属性：**元数据 metadata**、**规范 spec**和**状态 status**。

**元数据 metadata** 是用来标识 Kubernetes 对象的，每个对象都至少有3个元数据：namespace，name 和 uid。除此以外还有各种各样的标签 labels （自定义键值对）用来标识和匹配不同的对象，例如用户可以用标签 env 来标识区分不同的服务部署环境，分别用 env=dev、env=testing、env=production 来标识开发、测试、生产的不同服务；

**规范 spec** 描述了用户期望 Kubernetes 集群中的分布式系统达到的理想状态（Desired State），例如用户可以通过复制控制器 Replication Controller 设置期望的 Pod 副本数为3；

**状态 status** 描述了系统实际当前达到的状态（Status），例如系统当前实际的 Pod 副本数为2；那么复制控制器当前的程序逻辑就是自动启动新的 Pod，争取达到副本数为3。

Kubernetes 中所有的配置都是通过 API 对象的 spec 去设置的，也就是用户通过配置系统的理想状态来改变系统，这是 Kubernetes 重要设计理念之一，即所有的操作都是声明式（Declarative）的而不是命令式（Imperative）的。声明式操作在分布式系统中的好处是稳定，不怕丢操作或运行多次，例如设置副本数为3的操作运行多次也还是一个结果，而给副本数加1的操作就不是声明式的，运行多次结果就错了。

---

基本的 Kubernetes 对象包括：

* Pod
* Service
* Volume
* Namespace

<br>

<font size="5"><u>Pod</u></font>

Pod 是在 Kubernetes 集群中运行部署应用或服务的最小单元，它包含一个或多个容器。Pod 的设计理念是支持多个容器在一个 Pod 中共享网络地址和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。Pod 对多容器的支持是 Kubernetes 最基础的设计理念。