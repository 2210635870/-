---
layout: post
title: "Kubernetes 架构与设计"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### 1、Kubernetes 设计概述



### 2、Kubernetes 架构概述

Kubernetes 集群里分为主节点（master）与工作节点（Node）,工作节点是容器应用运行的地方。主节点负责管理整个集群，持续配置并维护集群保持期望的状态。

核心组件架构图：
![avatar](/images/k8s-old-arch.png)

Kubernetes 主要由以下几个核心组件组成：

---

*控制面板部分*

#### etcd:

保存整个集群的状态信息。同时因为有watch(观察者)的支持，各部件协调中的改变可以很快被察觉。

#### kube-apiserver:

提供了资源操作的唯一入口，并提供认证、授权、访问控制、API 注册和发现等机制。

#### kube-schedular:

负责资源的调度，按照预定的调度策略将 Pod 调度到相应的机器上。调度决策考虑的因素包括单个 Pod 和 Pod 集合的资源需求、硬件/软件/策略约束、亲和性和反亲和性规范、数据位置、工作负载间的干扰和最后时限。

#### kube-controller-manager:

负责维护集群的状态，比如故障检测、自动扩展、滚动更新等。这个组件包含一系列的控制器，但是为了降低复杂性，它们都被编译到同一个可执行文件，并在一个进程中运行。
这些控制器包括:

* 节点控制器（Node Controller）: 负责在节点出现故障时进行通知和响应。
* 副本控制器（Replication Controller）: 负责为系统中的每个副本控制器对象维护正确数量的 Pod。
* 端点控制器（Endpoints Controller）: 填充端点(Endpoints)对象(即加入 Service 与 Pod)。
* 服务帐户和令牌控制器（Service Account & Token Controllers）: 为新的命名空间创建默认帐户和 API 访问令牌。


一般地，kube-apiserver，kube-controller-manager 和 kube-schedular 都一起部署在主节点（master）, etcd 推荐独立部署，但也可以部署在主节点里。

---

*工作节点部分*

#### kubelet:

一个在集群中每个节点上运行的代理。它保证容器都运行在 Pod 中，负责维护容器的生命周期，同时也负责 Volume 和网络的管理。

kubelet 接收一组通过各类机制提供给它的 PodSpecs，确保这些 PodSpecs 中描述的容器处于运行状态且健康。kubelet 不会管理不是由 Kubernetes 创建的容器。

#### kube-proxy:

群中每个节点上运行的网络代理,实现 Kubernetes Service 概念的一部分，负责为 Service 提供 cluster 内部的服务发现和负载均衡。

kube-proxy 维护节点上的网络规则。这些网络规则允许从集群内部或外部的网络会话与 Pod 进行网络通信。如果操作系统提供了数据包过滤层并可用的话，kube-proxy会通过它来实现网络规则。否则，kube-proxy 仅转发流量本身。

#### Container runtime（CRI）:

负责镜像管理以及 Pod 和容器的真正运行（CRI）。

---

#### cloud controller manager:

后面为了让特定的云服务供应商代码和 Kubernetes 核心相互独立演化，产生了一个新的组件，云控制器管理器（cloud controller manager，CCM）。在以前的版本中，核心的 Kubernetes 代码依赖于特定云提供商的代码来实现功能。在将来的版本中，云供应商专有的代码应由云供应商自己维护，并与运行 Kubernetes 的云控制器管理器相关联。

cloud-controller-manager 仅运行云提供商特定的控制器循环。您必须在 kube-controller-manager 中禁用这些控制器循环，您可以通过在启动 kube-controller-manager 时将 --cloud-provider 参数设置为 external 来禁用控制器循环。

以下控制器具有云提供商依赖性:

节点控制器（Node Controller）: 用于检查云提供商以确定节点是否在云中停止响应后被删除
路由控制器（Route Controller）: 用于在底层云基础架构中设置路由
服务控制器（Service Controller）: 用于创建、更新和删除云提供商负载均衡器
数据卷控制器（Volume Controller）: 用于创建、附加和装载卷、并与云提供商进行交互以编排卷

云控制器管理器基于插件机制设计，允许新的云服务供应商通过插件轻松地与 Kubernetes 集成。目前已经有在 Kubernetes 上加入新的云服务供应商计划，并为云服务供应商提供从原先的旧模式迁移到新 CCM 模式的方案。

于是，kubernetes 的架构演变成如下：
![avatar](/images/k8s-new-arch.png)


<br>



