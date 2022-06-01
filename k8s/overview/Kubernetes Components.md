```
Addons： 扩展
outline: 概述 ; 略述 ; 显示…的轮廓 ; 勾勒…的外形 ; 梗概 ; 轮廓线 ; 略图
comprehensive: adj. 全部的;所有的;(几乎)无所不包的;详尽的;综合性的(接收各种资质的学生)
orchestration: n.编排，管弦乐编曲
arbitrary: adj.任意的
interfere: v. 干涉，冲突，妨碍
```

[TOC]



### Kubernetes Components

When you deploy Kubernetes, you get a cluster.

A Kubernetes cluster consists of a set of worker machines, called [nodes](https://kubernetes.io/docs/concepts/architecture/nodes/), that run containerized applications. Every cluster has at least one worker node.

The worker node(s) host the [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) that are the components of the application workload. The [control plane](https://kubernetes.io/docs/reference/glossary/?all=true#term-control-plane) manages the worker nodes and the Pods in the cluster. In production environments, the control plane usually runs across multiple computers and a cluster usually runs multiple nodes, providing fault-tolerance and high availability.

This document outlines the various components you need to have a complete and working Kubernetes cluster.

当你部署了Kubernetes，你就有了一个集群。

一个Kubernetes集群由一组工作机器组成，成为Nodes，在Node上面会运行容器化的应用。每一个集群至少需要一个工作Node。

工作节点托管着Pods组件，Pod组件作为应用程序workload。Control Plane管理集群中的工作节点和Pods。在生产环境中，Control Plane通常在多个计算机上运行，并且一个集群通常运行着多个Nodes，以此提供容错性和高可用性。

这篇文章概括了一些组件，这些组件是你运行一个完整的Kubernetes集群所必不可少的。

Here's the diagram of a Kubernetes cluster with all the components tied together.

![Components of Kubernetes](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)

#### Control Plane Components

The control plane's components make global decisions about the cluster (for example, scheduling), as well as detecting and responding to cluster events (for example, starting up a new [pod](https://kubernetes.io/docs/concepts/workloads/pods/) when a deployment's `replicas` field is unsatisfied).

Control plane components can be run on any machine in the cluster. However, for simplicity, set up scripts typically start all control plane components on the same machine, and do not run user containers on this machine. See [Building High-Availability Clusters](https://kubernetes.io/docs/admin/high-availability/) for an example multi-master-VM setup.

Control Plane组件负责对集群做全局的决策（比如调度），同时也负责检测和响应集群事件（比如当Deployment的replicas不满足时开启一个新的Pod）。

Control Plane内部的组件可以分离运行在集群的任何机器上。但是一般为了简单，设置脚本一般会把Control Plane的组件都运行在一个机器上，并且这台机器上上不会运行用户的容器。可以查看[Building High-Availability Clusters](https://kubernetes.io/docs/admin/high-availability/)来如果设置多个Master。



##### kube-apiserver

The API server is a component of the Kubernetes [control plane](https://kubernetes.io/docs/reference/glossary/?all=true#term-control-plane) that exposes the Kubernetes API. The API server is the front end for the Kubernetes control plane.

The main implementation of a Kubernetes API server is [kube-apiserver](https://kubernetes.io/docs/reference/generated/kube-apiserver/). kube-apiserver is designed to scale horizontally—that is, it scales by deploying more instances. You can run several instances of kube-apiserver and balance traffic between those instances.

API Server组件在Control Plane中负责暴露Kubernetes的API。API Server是Control Plane的前端组件。

Kubernetes API Server的主要实现就是[kube-apiserver](https://kubernetes.io/docs/reference/generated/kube-apiserver/)。Kube-apiserver被设计成可以水平扩容-那就是说可以通过部署多个实例来进行水平扩容。你可以运行多个kube-apiserver实例来将流量负载均衡到这些实例上。



##### etcd

Consistent and highly-available key value store used as Kubernetes' backing store for all cluster data.

If your Kubernetes cluster uses etcd as its backing store, make sure you have a [back up](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster) plan for those data.

You can find in-depth information about etcd in the official [documentation](https://etcd.io/docs/).

一致性以及高可用的KV存储组件，Kubernetes使用它来存储集群的数据。

如果你的Kubernetes集群使用etc的作为数据存储，确保你有一个数据备份的计划。



##### kube-scheduler

Control plane component that watches for newly created [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) with no assigned [node](https://kubernetes.io/docs/concepts/architecture/nodes/), and selects a node for them to run on.

Factors taken into account for scheduling decisions include: individual and collective resource requirements, hardware/software/policy constraints, affinity and anti-affinity specifications, data locality, inter-workload interference, and deadlines.

kube-scheduler组件负载观察新创建并且没有分配Node的Pods，并且为他们选择一个Node给Pod去运行。

影响调度决策的因素包括：单个和总Pod资源需求，硬件/软件/策略约束，亲和力与反亲和力的说明，数据所在位置，工作负载之间的干涉，截止日期。



##### kube-controller-manager

Control Plane component that runs [controller](https://kubernetes.io/docs/concepts/architecture/controller/) processes.

Logically, each [controller](https://kubernetes.io/docs/concepts/architecture/controller/) is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.

These controllers include:

- Node controller: Responsible for noticing and responding when nodes go down.
- Replication controller: Responsible for maintaining the correct number of pods for every replication controller object in the system.
- Endpoints controller: Populates the Endpoints object (that is, joins Services & Pods).
- Service Account & Token controllers: Create default accounts and API access tokens for new namespaces.

kube-controller-manager组件负载运行Controller进程。

逻辑上来说，每个Controller都是一个隔离的进程，但是为了减少复杂性，他们被编译到一个包并运行在一个进程中。

这些Controllers包括：

- Node Controller：当Node出现故障时候负责通知和响应
- Replication Controller： 负责为系统中Replication Conntroller Object的维护正确数量的Pods。
- Endpoints Controller： Populates the Endpoints object (that is, joins Services & Pods).
- Service Account & Token Controllers: 创建默认的账户和API访问Token给新的Namespaces。



##### cloud-controller-manager

A Kubernetes [control plane](https://kubernetes.io/docs/reference/glossary/?all=true#term-control-plane) component that embeds cloud-specific control logic. The cloud controller manager lets you link your cluster into your cloud provider's API, and separates out the components that interact with that cloud platform from components that just interact with your cluster.

The cloud-controller-manager only runs controllers that are specific to your cloud provider. If you are running Kubernetes on your own premises, or in a learning environment inside your own PC, the cluster does not have a cloud controller manager.

As with the kube-controller-manager, the cloud-controller-manager combines several logically independent control loops into a single binary that you run as a single process. You can scale horizontally (run more than one copy) to improve performance or to help tolerate failures.

The following controllers can have cloud provider dependencies:

- Node controller: For checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding
- Route controller: For setting up routes in the underlying cloud infrastructure
- Service controller: For creating, updating and deleting cloud provider load balancers

cloud-controller-manager组件嵌入了特定云平台的控制逻辑。cloud-controller-manager可以让你把你的Kubernetes集群跟云服务提供商的API连接起来，cloud-controller-manager把跟云平台交互的组件和负责跟你集群内部交互的组件进行了分离。

cloud-controller-manager只运行跟你特定云服务提供商相关的Controller。如果你只在自己环境中或者学习环境中运行集群，那么你不需要安装cloud-controller-manager组件。

和kube-controller-mananger类似cloud-controller-manager组件组合了一些逻辑独立的控制回路（controller）在一个单独的包中，所以你可以把它以一个单独的进程进行运行。你可以水平扩容来提高性能以及提高容错性。

如下的Controllers包含对云服务提供商的依赖：

- Node Controller： 用于在一个Node停止响应后通过云服务提供商来检查并决定是否该Node被删除了。
- Route Controller： 用于在云基础设施上设置路由
- Service Controller： 用于创建，更新，删除云服务提供商的负载均衡器。



#### Node Components

Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.

Node组件运行在每一个Node上，维护运行的Pods和提供Kubernetes运行时环境。



##### kubelet

An agent that runs on each [node](https://kubernetes.io/docs/concepts/architecture/nodes/) in the cluster. It makes sure that [containers](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/#why-containers) are running in a [Pod](https://kubernetes.io/docs/concepts/workloads/pods/).

The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. The kubelet doesn't manage containers which were not created by Kubernetes.

作为一个代理运行在集群的Node上。他用来确保容器运行在Pod中。

Kubelet负责接收一组Pod规格说明，这些PodSpecs通过各种机制提供，并且确保这些PodSpecs中描述的容器正在运行并且很健康。kubelet不会管理不由Kubernetes创建的容器。



##### kube-proxy

kube-proxy is a network proxy that runs on each [node](https://kubernetes.io/docs/concepts/architecture/nodes/) in your cluster, implementing part of the Kubernetes [Service](https://kubernetes.io/docs/concepts/services-networking/service/) concept.

[kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/) maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.

kube-proxy uses the operating system packet filtering layer if there is one and it's available. Otherwise, kube-proxy forwards the traffic itself.

kube-proxy是一个网络代理，它运行在集群中的每个节点上，实现了部分Service的概念。

kube-proxy维护Node的网络规则。这些网络规则允许集群内外的回话可以跟Node中的Pods进行通信。

kube-proxy使用操作系统提供的数据包过滤器层（如果有的话）并用他来实现网络规则，否则自己转发这些流量。



##### Container runtime

The container runtime is the software that is responsible for running containers.

Kubernetes supports several container runtimes: [Docker](https://docs.docker.com/engine/), [containerd](https://containerd.io/docs/), [CRI-O](https://cri-o.io/#what-is-cri-o), and any implementation of the [Kubernetes CRI (Container Runtime Interface)](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md).

容器运行时是软件负责运行容器。

Kubernetes支持多种容器运行时：[Docker](https://docs.docker.com/engine/), [containerd](https://containerd.io/docs/), [CRI-O](https://cri-o.io/#what-is-cri-o), 以及其他任何只要实现了 [Kubernetes CRI (Container Runtime Interface)](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md).



#### Addons 扩展

Addons use Kubernetes resources ([DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset), [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), etc) to implement cluster features. Because these are providing cluster-level features, namespaced resources for addons belong within the `kube-system` namespace.

扩展使用Kubernetes的资源(DaemonSet，Deployment，etc)来实现集群特性。因为他们提供了集群级别的特性，这些扩展都分配到一个特定的命名空间中,即`kube-system`。

Selected addons are described below; for an extended list of available addons, please see [Addons](https://kubernetes.io/docs/concepts/cluster-administration/addons/).



##### DNS domain name service 域名服务器

While the other addons are not strictly required, all Kubernetes clusters should have [cluster DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/), as many examples rely on it.

Cluster DNS is a DNS server, in addition to the other DNS server(s) in your environment, which serves DNS records for Kubernetes services.

Containers started by Kubernetes automatically include this DNS server in their DNS searches.

DNS是必须要有的一个Addons，其他扩展则不是严格必须的，因为很多东西要依赖于DNS扩展。

Cluster DNS是一个DNS服务，跟你环境中的其他DNS服务一样，负责解析kubernetes中的DNS记录。

Kubernetes中启动的容器都会自动包含这个DNS服务到他们的DNS搜索列表中。



##### Web UI (Dashboard)

[Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) is a general purpose, web-based UI for Kubernetes clusters. It allows users to manage and troubleshoot applications running in the cluster, as well as the cluster itself.

Dashboard是提供给Kubernetes集群的，一个基于Web的通用的UI界面。他允许用户对集群中的应用进行手动管理和故障排查，也支持对集群本身的操作。



##### Container Resource Monitoring

[Container Resource Monitoring](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/) records generic time-series metrics about containers in a central database, and provides a UI for browsing that data.

容器资源观察器记录通用的容器相关的时序指标到中央数据库中，并提供一个UI界面来浏览这些数据。



##### Cluster-level Logging

A [cluster-level logging](https://kubernetes.io/docs/concepts/cluster-administration/logging/) mechanism is responsible for saving container logs to a central log store with search/browsing interface.

集群级别的日志机制负责保存容器日志到中央日志仓库中，提供搜索浏览的接口。



