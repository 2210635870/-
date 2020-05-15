---
layout: post
title: "Kubernetes 架构与设计"
description: "Kubernetes 入门系列"
category: Kubernetes
tags: [kubernetes入门]
---
{% include JB/setup %}


### 1、Kubernetes 架构概述

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


一般地，kube-apiserver，kube-controller-manager 和 kube-schedular 都一起部署在主节点（master），因为在目前版本中，控制面板组件通过非安全（没有加密或认证）端口和集群的 kube-apiserver 通信。这个端口通常只在主节点的 localhost 接口暴露。etcd 可以选择独立部署，但也可以部署在主节点里。

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

于是，Kubernetes 的架构演变成如下：
![avatar](/images/k8s-new-arch.png)


<br>
<br>


### 2、Kubernetes 集群组件间的通信

Kubernetes 集群组件间的通信可以分为两类：**工作节点到主节点**和**主节点到工作节点**。

无论是哪种方式，kube-apiserver 都是作为流量的交汇点，是集群组件间通信的核心。集群内各个组件都通过它作为交互的中间媒介，并将信息存入 etcd。当需要获取和操作这些数据时，则通过它提供的 REST 接口来实现。
![avatar](/images/k8s-networks.png)



**工作节点到主节点**

---

所有从工作节点到主节点的通信路径都终止于 apiserver（其它主节点组件没有被设计为可暴露远程服务）。在一个典型的部署中，apiserver 被配置为在一个安全的 HTTPS 端口（443）上监听远程连接并启用一种或多种形式的客户端身份认证机制。一种或多种客户端身份认证机制应该被启用，特别是在允许使用 匿名请求 或 service account tokens 的时候。

应该使用集群的公共根证书开通节点，如此它们就能够基于有效的客户端凭据安全的连接 apiserver。例如：在一个默认的 GCE 部署中，客户端凭据以客户端证书的形式提供给 kubelet。

想要连接到 apiserver 的 Pods 可以使用一个 service account 安全的进行连接。这种情况下，当 Pods 被实例化时 Kubernetes 将自动的把公共根证书和一个有效的不记名令牌注入到 pod 里。kubernetes service （所有 namespaces 中）都配置了一个虚拟 IP 地址，用于转发（通过 kube-proxy）请求到 apiserver 的 HTTPS endpoint。

这样的结果使得从集群（在工作节点上运行的组件和 pods）到主节点的缺省连接操作模式默认被保护，能够在不可信或公网中运行。

---

**主节点到工作节点**

---

从主节点（apiserver）到集群有两种主要的通信路径。第一种是从 apiserver 到集群中每个节点上运行的 kubelet 进程。第二种是从 apiserver 通过它的代理功能到任何 node、pod 或者 service。

<br>

*apiserver -> kubelet*

从 apiserver 到 kubelet 的连接用于获取 pods 日志、连接（通过 kubectl）运行中的 pods，以及使用 kubelet 的端口转发功能。这些连接终止于 kubelet 的 HTTPS endpoint。

默认的，apiserver 不会验证 kubelet 的服务证书，这会导致连接遭到中间人攻击，因而在不可信或公共网络上是不安全的。

为了对这个连接进行认证，请使用 --kubelet-certificate-authority 标记给 apiserver 提供一个根证书捆绑，用于 kubelet 的服务证书。

如果这样不可能，又要求避免在不可信的或公共的网络上进行连接，请在 apiserver 和 kubelet 之间使用 SSH 隧道（但新版中已标记为 Deprecated）。

最后，应该启用 Kubelet 用户认证和/或权限认证来保护 kubelet API。


*apiserver -> nodes, pods, and services*

从 apiserver 到 node、pod 或者 service 的连接默认为纯 HTTP 方式，因此既没有认证，也没有加密。他们能够通过给 API URL 中的 node、pod 或 service 名称添加前缀 https: 来运行在安全的 HTTPS 连接上。但他们即不会认证 HTTPS endpoint 提供的证书，也不会提供客户端证书。这样虽然连接是加密的，但它不会提供任何完整性保证。这些连接目前还不能安全的在不可信的或公共的网络上运行。



### 3、Kubernetes 集群组件交互

<br>

#### kubelet 进程与 apiserver 的交互:

每个工作节点上的 kubelet 每隔一个时间周期就会调用一次 apiserver 的 REST 接口报告自身状态，apiserver 接收到这些信息后，将节点状态信息更新到 etcd 中。

kubelet 也通过 apiserver 的 Watch 接口监听 Pod 信息，如果监听到新的 Pod 的副本被调度绑定到本节点，则执行 Pod 对应的容器的创建和启动逻辑；如果监听到 Pod 对象被删除，则删除本节点上相应的 Pod 容器；如果监听到修改 Pod 信息，则 kubelet 监听变化后，会相应修改本节点的 Pod 容器。

<br>

#### kube-controller-manager 与 apiserver 的交互:

kube-controller-manager 中的 Node Controller 模块通过 API Server 提供的 Watch 接口，实时监控 Node 的信息，并做相应处理。

<br>

#### kube-scheduler 与 apiserver 的交互：

当 Scheduler 通过 apiserver 的 Watch 接口监听到新建 Pod 副本的信息后，它会检索所有符合该 Pod 要求的 Node 列表，开始执行 Pod 调度逻辑，调度成功后将 Pod 绑定到目标节点上。