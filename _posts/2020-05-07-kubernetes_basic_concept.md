---
layout: post
title: "Kubernetes 入门基本概念"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### 1、Kubernetes 是什么

套用官网文档，Kubernetes 是一个可移植的、可扩展的开源平台，用于管理容器化服务以及其工作负载，可促进服务的声明式配置和自动化配置。Kubernetes 拥有一个庞大且快速增长的生态系统。

<br>

### 2、Kubernetes 的意义

应用部署发展史：
![avatar](/images/container_evolution.svg)

**传统部署时代：** 早期，在物理服务器上运行应用程序。最明显的缺点是无法为物理服务器中的应用程序定义资源边界，这会导致资源分配问题。例如，如果在物理服务器上运行多个应用程序，则可能会出现一个应用程序占用大部分资源的情况，结果可能导致其他应用程序的性能下降。一种解决方案是在不同的物理服务器上运行每个应用程序，但是由于资源利用不足而无法扩展，并且组织维护许多物理服务器的成本很高。

**虚拟化部署时代：** 作为解决方案，引入了虚拟化功能，它允许您在单个物理服务器的 CPU 上运行多个虚拟机（VM）。虚拟化功能允许应用程序在 VM 之间隔离，并提供安全级别，因为一个应用程序的信息不能被另一应用程序自由地访问。

因为虚拟化可以轻松地添加或更新应用程序、降低硬件成本等等，所以虚拟化可以更好地利用物理服务器中的资源，并可以实现更好的可伸缩性。

每个 VM 是一台完整的计算机，在虚拟化硬件之上运行所有组件，包括其自己的操作系统。

**容器部署时代：** 容器类似于 VM，但是它们具有轻量级的隔离属性，可以在应用程序之间共享操作系统（OS）。因此，容器被认为是轻量级的。容器与 VM 类似，具有自己的文件系统、CPU、内存、进程空间等。由于它们与基础架构分离，因此可以跨云和 OS 分发进行移植。

容器因具有许多优势而变得流行起来，详情可参考[官网](https://kubernetes.io/zh/docs/concepts/overview/what-is-kubernetes/)。

<br>

容器是打包和运行应用程序的好方式。但要是在生产环境中推广使用还需要解决很多问题，例如，您需要管理运行应用程序的容器，并确保不会停机，如果一个容器发生故障，则需要启动另一个容器。

不同容器中的应用程序默认是网络隔离的，即容器间网络互不相通，特别是在跨主机集群环境中，需要做专门的网络配置来处理。

如由果系统来处理以上行为，会不会更美好？这就是 Kubernetes 的意义所在！Kubernetes 为您提供了一个可弹性运行分布式系统的框架。Kubernetes 会满足容器应用的扩展要求、故障转移、部署模式等。例如，Kubernetes 可以轻松管理系统的金丝雀部署。Kubernetes 负责打通集群内容器间的网络通讯，并实现负载均衡和分配网络流量。


<br>

### 3、Kubernetes 的组成

一个 Kubernetes 集群由 master 主节点和 node 工作节点组成，一个最小的 Kubernetes 集群至少要有一个主节点和一个工作节点。

主节点包含 Kubernetes 的主控组件。主控组件包含三个进程：kube-apiserver、kube-controller-manager 和 kube-scheduler。主控组件进程一般都跑在集群中一个单独的节点上。主节点负责维护 Kubernetes 集群的目标状态。

工作节点都是用来运行你的容器化应用和云工作流的机器。主节点控制所有工作节点；你很少需要和工作节点进行直接通信。工作节点都运行两个进程：

* kubelet，作用是和 master 节点进行通信。
* kube-proxy，一种网络代理，将 Kubernetes 的网络服务代理到每个节点上。

<br>

### 4、Kubernetes 对象

所谓 Kubernetes 对象，是用来表示系统状态的抽象概念，它们是持久化的实体。它们描述了如下信息：
* 哪些容器化应用在运行（以及在哪个 Node 上）
* 可以被应用使用的资源
* 关于应用运行时表现的策略，比如重启策略、升级策略，以及容错策略

具体举例包括：部署的容器化应用和负载、与它们相关的网络和磁盘资源以及有关集群运行的其他操作的信息。

Kubernetes 对象是 “目标性记录”，就是说一旦创建对象，本质就是告诉 Kubernetes 系统我期望要达成某些系统状态，然后 Kubernetes 就开始持续工作，以达到并一直维持期望状态。

每个 Kubernetes 对象包含两个嵌套的对象字段，它们负责管理对象的配置：对象 spec 和 对象 status 。 spec 是必需的，它描述了对象的期望状态（Desired State） —— 希望对象所具有的特征。 status 描述了对象的实际状态（Actual State） ，它是由 Kubernetes 系统提供和更新的。在任何时刻，Kubernetes 控制面一直努力地管理着对象的实际状态以与期望状态相匹配。

操作 Kubernetes 对象，无论是创建、修改，或者删除都需要通过 Kubernetes API（与 kube-apiserver 进程通讯）。

<br>

基本的 Kubernetes 对象包括：

* Pod
* Service
* Volume
* Namespace

Kubernetes 也包含大量的被称作 Controller 的高级抽象。控制器基于基本对象构建并提供额外的功能和方便使用的特性。具体包括:

* Deployment
* DaemonSet
* StatefulSet
* ReplicaSet
* Job

后文会有针对这些对象的具体讲解。

<br>

### 5、描述 Kubernetes 对象

当创建 Kubernetes 对象时，必须提供对象的规约，用来描述该对象的期望状态，以及关于对象的一些基本信息（例如名称）。 当使用 Kubernetes API 创建对象时（或者直接创建，或者基于kubectl），API 请求必须在请求体中包含 JSON 格式的信息。 **大多数情况下，需要在 .yaml 文件中为 kubectl 提供这些信息**。 kubectl 在发起 API 请求时，将这些信息转换成 JSON 格式。

一个展示了 Kubernetes Deployment 的必需字段和对象规约的示例 yaml 文件：
```
apiVersion: apps/v1    # 1.9.0 版本以前使用 apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2      # 表明按照下面 template 的规格，运行 2 个副本的的 pod
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

在 Kubernetes 对象对应的 .yaml 文件中，需要配置如下的字段：

* apiVersion —— 创建该对象所使用的 Kubernetes API 的版本。
* kind —— 想要创建的对象的类型。
* metadata —— 帮助识别对象唯一性的数据，包括一个 name 字符串、UID 和可选的 namespace。

您也需要提供对象的 spec 字段。对象 spec 的精确格式对每个 Kubernetes 对象来说是不同的，包含了特定于该对象的嵌套字段。参考 [Kubernetes API 文档](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/) 能够帮助我们找到任何我们想创建的对象的 spec 格式。

<br>

### 6、Kubernetes 控制面

Kubernetes 控制平面主要由 Kubernetes 主控组件和 kubelet 进程组成，管理整个 Kubernetes 集群。控制平面维护着系统中所有的 Kubernetes 对象的状态记录，并且通过连续的控制循环来管理这些对象的状态。在任意的给定时间点，控制面的控制环都能响应集群中的变化，并且让系统中所有对象的实际状态与你提供的预期状态相匹配。

比如， 当你通过 Kubernetes API 创建一个 Deployment 对象，你就为系统增加了一个新的目标状态。Kubernetes 控制平面记录着对象的创建，并启动必要的应用然后将它们调度至集群某个节点上来执行你的指令，以此来保持集群的实际状态和目标状态的匹配。