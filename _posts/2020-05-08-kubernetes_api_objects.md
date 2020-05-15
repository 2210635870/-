---
layout: post
title: "Kubernetes 核心技术概念"
description: "Kubernetes 入门系列"
category: Kubernetes
tags: [kubernetes入门]
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

Pod 是在 Kubernetes 集群中运行部署应用或服务的最小单元，它包含一个或多个容器。Pod 的设计理念是支持多个容器在一个 Pod 中共享网络地址和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。Pod 对多容器的支持是 Kubernetes 最基础的设计理念。比如你运行一个操作系统发行版的软件仓库，一个 Nginx 容器用来发布软件，另一个容器专门用来从源仓库做同步，这两个容器的镜像不太可能是一个团队开发的，但是他们一块儿工作才能提供一个微服务；这种情况下，不同的团队各自开发构建自己的容器镜像，在部署的时候组合成一个微服务对外提供服务。

Pod 是 Kubernetes 集群中所有业务类型的基础，可以看作运行在 Kubernetes 集群中的小机器人，不同类型的业务就需要不同类型的小机器人去执行。目前 Kubernetes 中的业务主要可以分为长期伺服型（long-running）、批处理型（batch）、节点后台支撑型（node-daemon）和有状态应用型（stateful application）；分别对应的小机器人控制器为 Deployment、Job、DaemonSet 和PetSet，本文后面会一一介绍。

<br>

<font size="5"><u>Service</u></font>

一个 Pod 只是一个运行服务的实例，随时可能在一个节点上停止，在另一个节点以一个新的 IP 启动一个新的 Pod，因此不能以确定的 IP 和端口号提供服务。要稳定地提供服务需要服务发现和负载均衡能力。服务发现完成的工作，是针对客户端访问的服务，找到对应的的后端服务实例。

在 Kubernetes 集群中，客户端需要访问的服务就是 Service 对象。每个 Service 会对应一个集群内部有效的虚拟 IP，集群内部通过虚拟 IP 访问一个服务。在 Kubernetes 集群中微服务的负载均衡是由 Kube-proxy 实现的。Kube-proxy 是 Kubernetes 集群内部的负载均衡器。它是一个分布式代理服务器，在 Kubernetes 的每个节点上都有一个；这一设计体现了它的伸缩性优势，需要访问服务的节点越多，提供负载均衡能力的 Kube-proxy 就越多，高可用节点也随之增多。

<br>

<font size="5"><u>Volume (存储卷)</u></font>

Kubernetes 集群中的存储卷跟 Docker 的存储卷有些类似，只不过 Docker 的存储卷作用范围为一个容器，而 Kubernetes 的存储卷的生命周期和作用范围是一个 Pod。每个 Pod 中声明的存储卷由 Pod 中的所有容器共享。

Kubernetes 支持非常多的存储卷类型，特别的，支持多种公有云平台的存储，包括 AWS，Google 和 Azure 云；支持多种分布式存储包括 GlusterFS 和 Ceph；也支持较容易使用的主机本地目录 hostPath 和 NFS。Kubernetes 还支持使用 Persistent Volume Claim 即 PVC 这种逻辑存储，使用这种存储，使得存储的使用者可以忽略后台的实际存储技术（例如 AWS，Google 或GlusterFS 和 Ceph），而将有关存储实际技术的配置交给存储管理员通过 Persistent Volume 来配置。

<br>

<font size="5"><u>Namespace（名字空间）</u></font>

名字空间为 Kubernetes 集群提供虚拟的隔离作用，Kubernetes 集群初始有两个名字空间，分别是默认名字空间 default 和系统名字空间 kube-system，除此以外，管理员可以可以创建新的名字空间满足需要。

<br>

---

核心的 Kubernetes 控制器基于基本对象构建并提供额外的功能和方便使用的特性。这些对象包括：

* ReplicaSet
* Deployment
* DaemonSet
* StatefulSet
* Job


<br>

<font size="5"><u>Replication Controller，RC（复制控制器）</u></font>

RC 是 Kubernetes 集群中最早的保证 Pod 高可用的 API 对象。通过监控运行中的 Pod 来保证集群中运行指定数目的 Pod 副本。指定的数目可以是多个也可以是1个；少于指定数目，RC 就会启动运行新的 Pod 副本；多于指定数目，RC 就会杀死多余的 Pod 副本。即使在指定数目为1的情况下，通过 RC 运行 Pod 也比直接运行 Pod 更明智，因为 RC 也可以发挥它高可用的能力，保证永远有1个 Pod 在运行。RC 是 Kubernetes 较早期的技术概念，只适用于长期伺服型的业务类型，比如控制小机器人提供高可用的 Web 服务。

<br>

<font size="5"><u>Replica Set，RS（副本集）</u></font>

RS 是新一代 RC，提供同样的高可用能力，区别主要在于 RS 后来居上，能支持更多种类的匹配模式。副本集对象一般不单独使用，而是作为 Deployment 的理想状态参数使用。

<br>

<font size="5"><u>Deployment（部署）</u></font>

部署表示用户对 Kubernetes 集群的一次更新操作。部署是一个比 RS 应用模式更广的 API 对象，可以是创建一个新的服务，更新一个新的服务，也可以是滚动升级一个服务。滚动升级一个服务，实际是创建一个新的 RS，然后逐渐将新 RS 中副本数增加到理想状态，将旧 RS 中的副本数减小到0的复合操作；这样一个复合操作用一个 RS 是不太好描述的，所以用一个更通用的 Deployment 来描述。以 Kubernetes 的发展方向，未来对所有长期伺服型的的业务的管理，都会通过 Deployment 来管理。


<br>

<font size="5"><u>DaemonSet（后台支撑服务集）</u></font>

长期伺服型和批处理型服务的核心在业务应用，可能有些节点运行多个同类业务的 Pod，有些节点上又没有这类 Pod 运行；而后台支撑型服务的核心关注点在 Kubernetes 集群中的节点（物理机或虚拟机），要保证每个节点上都有一个此类 Pod 运行。节点可能是所有集群节点，也可能是通过 nodeSelector 选定的一些特定节点。典型的后台支撑型服务包括，存储，日志和监控等在每个节点上支持 Kubernetes 集群运行的服务。

<br>

<font size="5"><u>StatefulSet（有状态服务集）</u></font>

StatefulSet 是用来管理有状态应用的工作负载 API 对象。StatefulSet 用来管理 Deployment 和扩展一组 Pod，并且能为这些 Pod 提供序号和唯一性保证。和 Deployment 相同的是，StatefulSet 管理了基于相同容器定义的一组 Pod。但和 Deployment 不同的是，StatefulSet 为它们的每个 Pod 维护了一个固定的 ID。这些 Pod 是基于相同的声明来创建的，但是不能相互替换：无论怎么调度，每个 Pod 都有一个永久不变的 ID。

StatefulSet 中的 Pod 一般都对应着稳定的、持久的存储卷。为 Pod 维护一个固定的 ID 的意义是将确定的 Pod 与确定的存储关联起来保证状态的连续性，如果一个Pod出现故障，从其他节点启动一个同样名字的Pod，要挂载上原来Pod的存储继续以它的状态提供服务。StatefulSet 的部署，缩放以及滚动更新时，Pod 调度或重调度是有序的，并且不会删除与 Pod 关联的存储卷。

<br>

<font size="5"><u>Job（任务）</u></font>

Job 是 Kubernetes 用来控制批处理型任务的 API 对象。批处理业务与长期伺服业务的主要区别是批处理业务的运行有头有尾，而长期伺服业务在用户不停止的情况下永远运行。Job 管理的 Pod 根据用户的设置把任务成功完成就自动退出了。成功完成的标志根据不同的 spec.completions 策略而不同：单 Pod 型任务有一个 Pod 成功就标志完成；定数成功型任务保证有  N个任务全部成功；工作队列型任务根据应用确认的全局成功而标志成功。