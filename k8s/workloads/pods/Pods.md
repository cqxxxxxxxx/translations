```
analogous to： 类似于
ephemeral: adj.短暂的
overall: adj. 全面的 ，总体的
cohesive： adj. 结成一个整体的;使结合的;使凝结的;使内聚的
disposable: adj.用后即丢弃的;一次性的;可动用的;可自由支配的
revised： v.修改 adj.改进的
separation of concerns： 关注点分离
constituent：n.(选区的)选民，选举人;成分;构成要素  adj.组成的;构成的
relevant： adj.有意义的
supervises： adj. 监督，治理
```

[TOC]



### Pods

*Pods* are the smallest deployable units of computing that you can create and manage in Kubernetes.

A *Pod* (as in a pod of whales or pea pod) is a group of one or more [containers](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/#why-containers), with shared storage/network resources, and a specification for how to run the containers. A Pod's contents are always co-located and co-scheduled, and run in a shared context. A Pod models an application-specific "logical host": it contains one or more application containers which are relatively tightly coupled. In non-cloud contexts, applications executed on the same physical or virtual machine are analogous to cloud applications executed on the same logical host.

As well as application containers, a Pod can contain [init containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) that run during Pod startup. You can also inject [ephemeral containers](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/) for debugging if your cluster offers this.



Pods是你在Kubernetes集群中所能创建和管理的最小的可部署的计算单元。

Pod是由一组共享存储与网络资源的容器（单个或多个）以及一个说明书用于指示怎么运行这些容器。同一个Pod中的内容总是一起部署，一起调度并且运行在一个共享的上下文中。Pod是一个模型，给应用定义了一个逻辑主机：他包含一个或多个紧密耦合的应用容器。运行在在非云环境中，即同一物理机或者虚拟机的应用，它其实跟在云环境中运行于同一逻辑主机中的应用是类似的。

和应用容器一样，Pod可以包含一个初始化容器在Pod启动的时候.你也可以注入一个短暂的容器用于debug（如果你的集群支持的话）。



#### What is a Pod?

> **Note:** While Kubernetes supports more [container runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes) than just Docker, [Docker](https://www.docker.com/) is the most commonly known runtime, and it helps to describe Pods using some terminology from Docker.

The shared context of a Pod is a set of Linux namespaces, cgroups, and potentially other facets of isolation - the same things that isolate a Docker container. Within a Pod's context, the individual applications may have further sub-isolations applied.

In terms of Docker concepts, a Pod is similar to a group of Docker containers with shared namespaces and shared filesystem volumes.

Pod的共享上下文包括一组Linux namespaces，cgroups和其他潜在的隔离方面-这就跟Docker容器用来隔离的技术一样。在Pod上下文中，每个独立的应用可能会采用更进一步的隔离机制。

从Docker概念的角度来说，Pod就像是一组Docker容器，他们之间共享namespaces和文件系统，挂载卷。



#### Using Pods

Usually you don't need to create Pods directly, even singleton Pods. Instead, create them using workload resources such as [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) or [Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/). If your Pods need to track state, consider the [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) resource.

通常你不需要去直接创建Pods,即使是单例Pods。取而代之的是使用工作负载资源（workload resources）例如Deployment或者Job来创建Pods。如果你的Pods需要追踪状态，考虑使用Stateful资源。

Pods in a Kubernetes cluster are used in two main ways:

- **Pods that run a single container**. The "one-container-per-Pod" model is the most common Kubernetes use case; in this case, you can think of a Pod as a wrapper around a single container; Kubernetes manages Pods rather than managing the containers directly.

- **Pods that run multiple containers that need to work together**. A Pod can encapsulate an application composed of multiple co-located containers that are tightly coupled and need to share resources. These co-located containers form a single cohesive unit of service—for example, one container serving data stored in a shared volume to the public, while a separate *sidecar* container refreshes or updates those files. The Pod wraps these containers, storage resources, and an ephemeral network identity together as a single unit.

  > **Note:** Grouping multiple co-located and co-managed containers in a single Pod is a relatively advanced use case. You should use this pattern only in specific instances in which your containers are tightly coupled.
  >
  > 把多个同位置，共同管理的容器组成一个Pod是相对比较高级的用例。你应该在你容器需要紧密耦合的场景下才使用这种方式。

Each Pod is meant to run a single instance of a given application. If you want to scale your application horizontally (to provide more overall resources by running more instances), you should use multiple Pods, one for each instance. In Kubernetes, this is typically referred to as *replication*. Replicated Pods are usually created and managed as a group by a workload resource and its [controller](https://kubernetes.io/docs/concepts/architecture/controller/).

See [Pods and controllers](https://kubernetes.io/docs/concepts/workloads/pods/#pods-and-controllers) for more information on how Kubernetes uses workload resources, and their controllers, to implement application scaling and auto-healing.

Pods在Kubernetes中主要有两个作用:

- Pods运行一个单一的容器: 一个容器一个Pod是最常用的方式；在这种情况下你可以把Pod当做一个Contrainer，但是Kubernetes不会直接操作Container来进行Pod的管理。
- Pods运行一组需要互相协作的容器: Pod可以封装由多个紧密耦合共享资源的容器到一个应用中。这些处于同一位置的容器构成了一个单一的紧密结合的单元服务。比如一个容器负责对外提供基于共享volume中数据的服务，与此同时另一个分离的sidecar容器则来刷新和更新这些数据。Pod则把这两个容器，以及存储资源和网络资源打包成一个单元。

每个Pod都用于运行应用的一个实例。如果你想要水平扩展你的应用（通过运行多个实例来提供更全面的资源），你应该使用多个Pods，每个Pod运行一个实例。在Kubernetes中，这通常叫做replication。Replicated Pods通常通过工作负载和他们的Controller来创建和管理。



##### How Pods manage multiple containers（Pods如何管理多个容器）

Pods are designed to support multiple cooperating processes (as containers) that form a cohesive unit of service. The containers in a Pod are automatically co-located and co-scheduled on the same physical or virtual machine in the cluster. The containers can share resources and dependencies, communicate with one another, and coordinate when and how they are terminated.

For example, you might have a container that acts as a web server for files in a shared volume, and a separate "sidecar" container that updates those files from a remote source, as in the following diagram:

Some Pods have [init containers](https://kubernetes.io/docs/reference/glossary/?all=true#term-init-container) as well as [app containers](https://kubernetes.io/docs/reference/glossary/?all=true#term-app-container). Init containers run and complete before the app containers are started.

Pods natively provide two kinds of shared resources for their constituent containers: [networking](https://kubernetes.io/docs/concepts/workloads/pods/#pod-networking) and [storage](https://kubernetes.io/docs/concepts/workloads/pods/#pod-storage).

Pods被设计成支持多个相互合作的进程(as容器),这些进程组成一个紧密结合的服务单元。在Pod中的这些容器都会被集群自动的统一放置和调度到同一个物理机或者虚拟机上。这些容器可以共享资源和依赖，互相通信和协作，以及何时或者如何终止自己。

比如你有一个容器基于共享的volume中的文件作为一个Web Server提供服务，而另一个单独的sidecar容器则用于从远程数据源更新这些文件，如下图：https://d33wubrfki0l68.cloudfront.net/aecab1f649bc640ebef1f05581bfcc91a48038c4/728d6/images/docs/pod.svg

有些Pods具有初始化容器，用于在应用容器启动之前运行和执行。

Pods天生的为内部的容器提供了两种共享资源：网络和存储。



#### Working with Pods 

You'll rarely create individual Pods directly in Kubernetes—even singleton Pods. This is because Pods are designed as relatively ephemeral, disposable entities. When a Pod gets created (directly by you, or indirectly by a [controller](https://kubernetes.io/docs/concepts/architecture/controller/)), the new Pod is scheduled to run on a [Node](https://kubernetes.io/docs/concepts/architecture/nodes/) in your cluster. The Pod remains on that node until the Pod finishes execution, the Pod object is deleted, the Pod is *evicted* for lack of resources, or the node fails.

> **Note:** Restarting a container in a Pod should not be confused with restarting a Pod. A Pod is not a process, but an environment for running container(s). A Pod persists until it is deleted.
>
> 容器的重启和Pod的重启不是一个概念上的不要混淆。Pod不是进程，他只是运行容器的环境。Pod会持续存在直到他被删除。

When you create the manifest for a Pod object, make sure the name specified is a valid [DNS subdomain name](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

极少数情况下你会直接创建Pods，即使是单实例Pods。这是因为Pods被设计于相对临时的，用后即抛的实体。当创建了一个Pod（你直接创建或者间接通过Controller创建），这个新的Pod会被调度运行在你集群中的一个Node上面。这个Pod会一直呆在这个Node上直到Pod完成他执行的任务，或者Pod Object被删除了，或者Pod因为资源短缺被驱逐了，或者所在Node故障挂了。

当你创建一个Pod Object的清单时，确保name是一个合法的DNS子域名。



##### Pods and controllers

You can use workload resources to create and manage multiple Pods for you. A controller for the resource handles replication and rollout and automatic healing in case of Pod failure. For example, if a Node fails, a controller notices that Pods on that Node have stopped working and creates a replacement Pod. The scheduler places the replacement Pod onto a healthy Node.

Here are some examples of workload resources that manage one or more Pods:

- [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset)

你可以使用workload资源来为你创建和管理多个Pods。一个资源Controller负责在Pod故障时候进行副本，推出和自动回复。比如，如果一个Node故障了，Controller发现该Node上的Pods停止工作了，那么就会自动创建一个替代Pod，scheduler则会自动把这个替代Pod调度到一个健康的Node上。

这里是一些工作负载资源的例子，他们可以管理一个或多个Pods。1. Deployment 2.StatefulSet 3.DaemonSet



##### Pod templates

Controllers for [workload](https://kubernetes.io/docs/concepts/workloads/) resources create Pods from a *pod template* and manage those Pods on your behalf.

PodTemplates are specifications for creating Pods, and are included in workload resources such as [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [Jobs](https://kubernetes.io/docs/concepts/jobs/run-to-completion-finite-workloads/), and [DaemonSets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/).

Each controller for a workload resource uses the `PodTemplate` inside the workload object to make actual Pods. The `PodTemplate` is part of the desired state of whatever workload resource you used to run your app.

工作负载的Controller负责基于一个Pod模板创建并管理Pods。

PodTemplates包含创建工作负载资源的创建说明，比如[Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [Jobs](https://kubernetes.io/docs/concepts/jobs/run-to-completion-finite-workloads/), and [DaemonSets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)。

工作负载的控制器会使用负载对象中的 `PodTemplate` 来生成实际的 Pod。 `PodTemplate` 是你用来运行应用时指定的负载资源的目标状态的一部分。

下面这个示例清单是一个简单的Job，他有一个template用于开启一个容器。这个Pod中的容器会先打印一个信息然后终止。

The sample below is a manifest for a simple Job with a `template` that starts one container. The container in that Pod prints a message then pauses.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    # This is the pod template
    spec:
      containers:
      - name: hello
        image: busybox
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
      restartPolicy: OnFailure
    # The pod template ends here
```

Modifying the pod template or switching to a new pod template has no effect on the Pods that already exist. Pods do not receive template updates directly. Instead, a new Pod is created to match the revised pod template.

修改这个Pod template或者切换一个新的Pod template不会影响已经存在的Pods。Pods不会因为template的更新而直接发生变动更新。取而代之的是创建一个新的Pod来匹配这个修改后的Pod模板。

For example, the deployment controller ensures that the running Pods match the current pod template for each Deployment object. If the template is updated, the Deployment has to remove the existing Pods and create new Pods based on the updated template. Each workload resource implements its own rules for handling changes to the Pod template.

比如说，Deployment Controller针对每个Deployment Object会确保运行中的Pods匹配他们当前的Pod template。如果模板更新了，Deployment需要去删除存在的Pods并且基于更新后的template来创建新的Pods。每一种工作负载资源都会实现他们自己的规则来处理这种Pod Template更新的情况。

On Nodes, the [kubelet](https://kubernetes.io/docs/reference/generated/kubelet) does not directly observe or manage any of the details around pod templates and updates; those details are abstracted away. That abstraction and separation of concerns simplifies system semantics, and makes it feasible to extend the cluster's behavior without changing existing code.

在Nodes上，Kubelet不会直接观察或者管理Pod template的相关细节和修改更新；这些细节都被抽象出来了。这种抽象和关注点分离简化了整个系统的语义，并且使得用户可以在不改变现有代码的前提下就能扩展集群的行为。



#### Resource sharing and communication

Pods enable data sharing and communication among their constituent containers. 

Pods支持数据共享和彼此通信在构成它的容器之间。

##### Storage in Pods

A Pod can specify a set of shared storage [volumes](https://kubernetes.io/docs/concepts/storage/volumes/). All containers in the Pod can access the shared volumes, allowing those containers to share data. Volumes also allow persistent data in a Pod to survive in case one of the containers within needs to be restarted. See [Storage](https://kubernetes.io/docs/concepts/storage/) for more information on how Kubernetes implements shared storage and makes it available to Pods.

一个Pod可以致命一组共享的存储volumes。所有该Pod中的容器都可以访问共享的volumes，允许这些容器去共享数据。Volumes也支持持久化数据来提供为容器提供支持，以便万一什么时候容器需要重启可以恢复数据。查看 [Storage](https://kubernetes.io/docs/concepts/storage/)来学习更多信息关于Kubernetes如何实现共享的存储和如何把它提供给Pods。



##### Pod networking

Each Pod is assigned a unique IP address for each address family. Every container in a Pod shares the network namespace, including the IP address and network ports. Inside a Pod (and **only** then), the containers that belong to the Pod can communicate with one another using `localhost`. When containers in a Pod communicate with entities *outside the Pod*, they must coordinate how they use the shared network resources (such as ports). Within a Pod, containers share an IP address and port space, and can find each other via `localhost`. The containers in a Pod can also communicate with each other using standard inter-process communications like SystemV semaphores or POSIX shared memory. Containers in different Pods have distinct IP addresses and can not communicate by IPC without [special configuration](https://kubernetes.io/docs/concepts/policy/pod-security-policy/). Containers that want to interact with a container running in a different Pod can use IP networking to communicate.

Containers within the Pod see the system hostname as being the same as the configured `name` for the Pod. There's more about this in the [networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/) section.

每个Pod都会在每个address family中分配一个唯一的IP地址。每个在Pod的中的容器都共享网络namespace，包括IP地址和网络端口.在Pod中（只有在这种情况下），内部的容器可以采用localhost来相互通信。当Pod中的这些容器想要跟Pod外的实体通信时，这些内部的容器需要相互协调如何使用分配这些共享的网络资源（比如端口）。在同一个Pod内部，容器会共享IP地址和端口，然后可以通过localhost相互通信。Pod内的容器也可以通过使用标准的进程间通信比如SystemV semaphores or POSIX shared memory来进行相互通信。容器在不同的Pod中有不同的IP地址并且无法通过IPC的方式进行通信（除非进行特殊的配置）。运行在不同Pod中的容器如果想要相互通信可以使用IP联网的方式。

Pod 中的容器所看到的系统主机名与为 Pod 配置的 `name` 属性值相同。 [网络](https://kubernetes.io/zh/docs/concepts/cluster-administration/networking/)部分提供了更多有关此内容的信息。



#### Privileged mode for containers 容器的权限模式[ ](https://kubernetes.io/docs/concepts/workloads/pods/#privileged-mode-for-containers)

Any container in a Pod can enable privileged mode, using the `privileged` flag on the [security context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) of the container spec. This is useful for containers that want to use operating system administrative capabilities such as manipulating the network stack or accessing hardware devices. Processes within a privileged container get almost the same privileges that are available to processes outside a container.

任何在Pod中的容器可以开启权限模式，通过在容器说明配置的security context模块中使用privileged参数。这是很有用的如果容器想要使用OS管理员权限，比如修改网络栈或者访问硬件设备。进程在带有权限的容器中几乎跟运行在容器外的进程有同样的权限。

> **Note:** Your [container runtime](https://kubernetes.io/docs/setup/production-environment/container-runtimes) must support the concept of a privileged container for this setting to be relevant.
>
> 你的容器运行时必须支持权限容器的概念，所以这个设置才能够有意义，不然不生效。



#### Static Pods

*Static Pods* are managed directly by the kubelet daemon on a specific node, without the [API server](https://kubernetes.io/docs/concepts/overview/components/#kube-apiserver) observing them. Whereas most Pods are managed by the control plane (for example, a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)), for static Pods, the kubelet directly supervises each static Pod (and restarts it if it fails).

Static Pods are always bound to one [Kubelet](https://kubernetes.io/docs/reference/generated/kubelet) on a specific node. The main use for static Pods is to run a self-hosted control plane: in other words, using the kubelet to supervise the individual [control plane components](https://kubernetes.io/docs/concepts/overview/components/#control-plane-components).

The kubelet automatically tries to create a [mirror Pod](https://kubernetes.io/docs/reference/glossary/?all=true#term-mirror-pod) on the Kubernetes API server for each static Pod. This means that the Pods running on a node are visible on the API server, but cannot be controlled from there.

静态Pods会在一个特别的Node上由Kubelet daemon线程直接进行管理，API Server不会去关注这些Pods。尽管大多数Pods是由Control Plane管理的（比如Deloyment），对于静态Pods，kubelet之间监督治理这些Pods，如果挂掉了则会重启它。

Static Pods会绑定到特定的一个Node的kubelet上。Static Pods的主要用途是运行一个自托管的控制面板：换句话说，使用kubelet来治理独立的control plane的一些组件。

Kubelet自动的尝试为每一个Static Pod在API Server上创建一个镜像的Pod。这意味着运行在node上的Pods对于API Server是可见的，但是不由API Server管理。

