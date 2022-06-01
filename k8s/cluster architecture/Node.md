```
distros: 发行版
conjunction: 结合 in conjunction with
constrain: vt. 驱使，强迫
taints: v.污染 n.污点
Lease: n.租约
exponential: adj.指数的
```

[TOC]

## Nodes

Kubernetes runs your workload by placing containers into Pods to run on *Nodes*. A node may be a virtual or physical machine, depending on the cluster. Each node contains the services necessary to run [Pods](https://kubernetes.io/docs/concepts/workloads/pods/), managed by the [control plane](https://kubernetes.io/docs/reference/glossary/?all=true#term-control-plane).

Typically you have several nodes in a cluster; in a learning or resource-limited environment, you might have just one.

The [components](https://kubernetes.io/docs/concepts/overview/components/#node-components) on a node include the [kubelet](https://kubernetes.io/docs/reference/generated/kubelet), a [container runtime](https://kubernetes.io/docs/setup/production-environment/container-runtimes), and the [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/).

Kubernetes通过把容器放到Nodes中的Pods里去运行你的工作。一个Node可能是虚拟机或者物理机，这取决于你的集群。每一个Node都包含着运行Node的必要的服务，这些服务由Control Plane来管理。

通常你有若干个节点在集群中；在学习或者资源有限的环境中，你可能只有一个节点。

Node中的Components包括Kubelet，Container Runtime以及Kube-Proxy。

### Management

There are two main ways to have Nodes added to the [API server](https://kubernetes.io/docs/concepts/overview/components/#kube-apiserver):

1. The kubelet on a node self-registers to the control plane
2. You, or another human user, manually add a Node object

主要有两种方式来添加Node到API  Server上。

1. Node中的Kubelet把自己主动注册到Control Plane上
2. 你或者其他用户手动添加一个Node Object

After you create a Node object, or the kubelet on a node self-registers, the control plane checks whether the new Node object is valid. For example, if you try to create a Node from the following JSON manifest:

```json
{
  "kind": "Node",
  "apiVersion": "v1",
  "metadata": {
    "name": "10.240.79.157",
    "labels": {
      "name": "my-first-k8s-node"
    }
  }
}
```

Kubernetes creates a Node object internally (the representation). Kubernetes checks that a kubelet has registered to the API server that matches the `metadata.name` field of the Node. If the node is healthy (if all necessary services are running), it is eligible to run a Pod. Otherwise, that node is ignored for any cluster activity until it becomes healthy.

The name of a Node object must be a valid [DNS subdomain name](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

Kubernetes创建一个Node Object在内部（来代表这个Node)。Kubernetes会检查注册到API Server上并且metadata.name是匹配的Kubelet。如果这个Node是健康的(如果所有必要的服务都在运行),那么这个Node就是有资格去运行Pod，否则这个Node会被忽略直到他恢复健康。

Node Object的name必须是一个合法的DNS subdomain name。



####  [Self-registration of Nodes 自我注册](https://kubernetes.io/docs/concepts/architecture/nodes/#self-registration-of-nodes)

When the kubelet flag `--register-node` is true (the default), the kubelet will attempt to register itself with the API server. This is the preferred pattern, used by most distros.

当Kubelet的flag`--register-node`设置为true（默认就是true）时候，kubelet会尝试注册它自己到API Server上。在大多数发行版上，这是一个默认首选的方式。

For self-registration, the kubelet is started with the following options:

- `--kubeconfig` - Path to credentials to authenticate itself to the API server. 

- `--cloud-provider` - How to talk to a [cloud provider](https://kubernetes.io/docs/reference/glossary/?all=true#term-cloud-provider) to read metadata about itself.

- `--register-node` - Automatically register with the API server.

- `--register-with-taints` - Register the node with the given list of [taints](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) (comma separated `<key>=<value>:<effect>`).

  No-op if `register-node` is false.

- `--node-ip` - IP address of the node.

- `--node-labels` - [Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels) to add when registering the node in the cluster (see label restrictions enforced by the [NodeRestriction admission plugin](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#noderestriction)).

- `--node-status-update-frequency` - Specifies how often kubelet posts node status to master.

When the [Node authorization mode](https://kubernetes.io/docs/reference/access-authn-authz/node/) and [NodeRestriction admission plugin](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#noderestriction) are enabled, kubelets are only authorized to create/modify their own Node resource.

在自我注册模式下，kubelet会以一下参数进行启动：

- `--kubeconfig` - 证书的路径，用于向API Server表明自己的身份（证明我是我）。

- `--cloud-provider` - How to talk to a [cloud provider](https://kubernetes.io/docs/reference/glossary/?all=true#term-cloud-provider) to read metadata about itself.

- `--register-node` - 自动把自己注册到API Server上。

- `--register-with-taints` - Register the node with the given list of [taints](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) (comma separated `<key>=<value>:<effect>`).

  如果 `register-node` 是false时候不激活。

- `--node-ip` - Node的IP

- `--node-labels` - Node注册到集群时需要添加的labels(see label restrictions enforced by the [NodeRestriction admission plugin](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#noderestriction))

- `--node-status-update-frequency` - 表明Node向集群更新自己状态的频率

当[Node authorization mode](https://kubernetes.io/docs/reference/access-authn-authz/node/) and [NodeRestriction admission plugin](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#noderestriction) 启用时候，只授权Kubelet去创建修改它自己的Node资源。



#### [Manual Node administration 手动节点管理](https://kubernetes.io/docs/concepts/architecture/nodes/#manual-node-administration)

You can create and modify Node objects using [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/).

When you want to create Node objects manually, set the kubelet flag `--register-node=false`.

You can modify Node objects regardless of the setting of `--register-node`. For example, you can set labels on an existing Node, or mark it unschedulable.

You can use labels on Nodes in conjunction with node selectors on Pods to control scheduling. For example, you can constrain a Pod to only be eligible to run on a subset of the available nodes.

Marking a node as unschedulable prevents the scheduler from placing new pods onto that Node, but does not affect existing Pods on the Node. This is useful as a preparatory step before a node reboot or other maintenance.

To mark a Node unschedulable, run:

```shell
kubectl cordon $NODENAME
```

你可以通过kubectl来创建和修改Node Objects。

当你想手动创建Node Objects，设置kubelet参数为 `--register-node=false`.

你可以无视 `--register-node`参数对Node Objects进行修改。比如你可以在存在的Node上设置Labels或者标记这个Node为Unschedulable。

你可以使用Nodes上的Label并结合Pods上的Node Selectors来控制调度。比如您可以强迫Pod只运行在一部分有效的匹配的Nodes上面。

标记Node为不可调度可以阻止scheduler把新的Pod调度到该Node上面，但这不会影响之前存在的Pods。这在一个Node需要reboot或者维护时是很有用的一个前置步骤。

执行`kubectl cordon $NODENAME`可以标记一个Node为unschedulable。



### Node status[ ](https://kubernetes.io/docs/concepts/architecture/nodes/#node-status)

A Node's status contains the following information:

- [Addresses](https://kubernetes.io/docs/concepts/architecture/nodes/#addresses) 地址
- [Conditions](https://kubernetes.io/docs/concepts/architecture/nodes/#condition) 状态
- [Capacity and Allocatable](https://kubernetes.io/docs/concepts/architecture/nodes/#capacity) 容量和可分配
- [Info](https://kubernetes.io/docs/concepts/architecture/nodes/#info) 信息

You can use `kubectl` to view a Node's status and other details:

```shell
kubectl describe node <insert-node-name-here>  #该命令查看Node的status
```

#### Addresses

The usage of these fields varies depending on your cloud provider or bare metal configuration.

- HostName: The hostname as reported by the node's kernel. Can be overridden via the kubelet `--hostname-override` parameter.
- ExternalIP: Typically the IP address of the node that is externally routable (available from outside the cluster).
- InternalIP: Typically the IP address of the node that is routable only within the cluster.

这些字段的使用取决于你的云服务提供商或者裸金属服务器配置

- HostName：hostname由Node的内核设置,也可以通过kubelet的`--hostname-override` 参数来覆盖掉

- ExternanlIP: 通常这个IP地址是Node的外部可路由的地址（集群外可以通过这个IP访问到该Node）公网IP

- InternalIP：通常这个IP地址是这个节点只允许在集群内访问的地址。内网IP

  

#### Conditions

The `conditions` field describes the status of all `Running` nodes. Examples of conditions include:

| Node Condition       | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| `Ready`              | `True` if the node is healthy and ready to accept pods, `False` if the node is not healthy and is not accepting pods, and `Unknown` if the node controller has not heard from the node in the last `node-monitor-grace-period` (default is 40 seconds) |
| `DiskPressure`       | `True` if pressure exists on the disk size--that is, if the disk capacity is low; otherwise `False` |
| `MemoryPressure`     | `True` if pressure exists on the node memory--that is, if the node memory is low; otherwise `False` |
| `PIDPressure`        | `True` if pressure exists on the processes—that is, if there are too many processes on the node; otherwise `False` |
| `NetworkUnavailable` | `True` if the network for the node is not correctly configured, otherwise `False` |

If the Status of the Ready condition remains `Unknown` or `False` for longer than the `pod-eviction-timeout` (an argument passed to the [kube-controller-manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)), all the Pods on the node are scheduled for deletion by the node controller. The default eviction timeout duration is **five minutes**. In some cases when the node is unreachable, the API server is unable to communicate with the kubelet on the node. The decision to delete the pods cannot be communicated to the kubelet until communication with the API server is re-established. In the meantime, the pods that are scheduled for deletion may continue to run on the partitioned node.

如果Condition的Ready状态为Unknown或者False并持续了比`pod-eviction-timeout`设置的时间还长，那么所有在该Node上的Pods都会被Node Controller计划删除。默认的timeout设置时间为5分钟。在某些情况下，当Node不可及时，API Server不能跟Node上的Kubelet进行通讯，那么删除该Node上Pods的决定将发送不到Node的kubelet上直到连接重新建立。在这个无法通信的期间，被计划删除的Pods会持续在这些发生了网络分区的Node上运行。

The node controller does not force delete pods until it is confirmed that they have stopped running in the cluster. You can see the pods that might be running on an unreachable node as being in the `Terminating` or `Unknown` state. In cases where Kubernetes cannot deduce from the underlying infrastructure if a node has permanently left a cluster, the cluster administrator may need to delete the node object by hand. Deleting the node object from Kubernetes causes all the Pod objects running on the node to be deleted from the API server, and frees up their names.

Node Controller不会强制删除Pods直到确认Pods在集群中已经停止了运行。因此你也许可以看到处于 `Terminating` or `Unknown`状态的Pods运行在不可达的Node上。在某些情况下，如果Kubernetes无法根据下游的基础设置来决定是否一个Node已经永远的从集群中移除了，那么集群维护者需要手动的删除这个Node Object。从Kubernetes中删除Node Object会将这个Node中所有Pods从API Server中删除并释放他们的names。

The node lifecycle controller automatically creates [taints](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) that represent conditions. The scheduler takes the Node's taints into consideration when assigning a Pod to a Node. Pods can also have tolerations which let them tolerate a Node's taints.

See [Taint Nodes by Condition](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/#taint-nodes-by-condition) for more details.

Node的生命周期Controller会自动创建taints代表这个Node的Conditions。调度器会在为Pod分配Node时考虑这个Node的taints。Pods也可以通过Tolerations配置来允许自己忍受Node的taints。



#### Capacity and Allocatable[ ](https://kubernetes.io/docs/concepts/architecture/nodes/#capacity)

Describes the resources available on the node: CPU, memory and the maximum number of pods that can be scheduled onto the node.

The fields in the capacity block indicate the total amount of resources that a Node has. The allocatable block indicates the amount of resources on a Node that is available to be consumed by normal Pods.

You may read more about capacity and allocatable resources while learning how to [reserve compute resources](https://kubernetes.io/docs/tasks/administer-cluster/reserve-compute-resources/#node-allocatable) on a Node.

描述Node上资源的可用情况：CPU,memory以及Node上最大可支持的Pods数量。

Capactiy指明Node上总的资源数量。Allocatable则指明剩余可被Pods使用的资源数量。

你也许会在学习如何在Node上预留计算资源时候学到更多有关Capacity和Allocatable的知识。



#### Info

Describes general information about the node, such as kernel version, Kubernetes version (kubelet and kube-proxy version), Docker version (if used), and OS name. This information is gathered by Kubelet from the node.

描述Node一些常规的信息，比如Kernel的Version，Kubernetes Version(kubelet and kube-proxy version)，Docker version（如果使用了Docker）以及OS name。这些信息是由Kubelet在Node上收集到的。



#### Node controller[ ](https://kubernetes.io/docs/concepts/architecture/nodes/#node-controller)

The node [controller](https://kubernetes.io/docs/concepts/architecture/controller/) is a Kubernetes control plane component that manages various aspects of nodes.

The node controller has multiple roles in a node's life. The first is assigning a CIDR block to the node when it is registered (if CIDR assignment is turned on).

The second is keeping the node controller's internal list of nodes up to date with the cloud provider's list of available machines. When running in a cloud environment, whenever a node is unhealthy, the node controller asks the cloud provider if the VM for that node is still available. If not, the node controller deletes the node from its list of nodes.

The third is monitoring the nodes' health. The node controller is responsible for updating the NodeReady condition of NodeStatus to ConditionUnknown when a node becomes unreachable (i.e. the node controller stops receiving heartbeats for some reason, for example due to the node being down), and then later evicting all the pods from the node (using graceful termination) if the node continues to be unreachable. (The default timeouts are 40s to start reporting ConditionUnknown and 5m after that to start evicting pods.) The node controller checks the state of each node every `--node-monitor-period` seconds.

Node Controller是Kubenetes Control Plane的一个组件，用于管理Node的总多方面。

Node Controller在Node的生命中起着若干个作用。

第一是当Node注册上来时候回分配一个CIDR block给该Node（如果CIDR分配为开启状态）

第二是维护Node Controller内部的Node列表跟云服务提供商的可用机器列表一致。当运行在云环境中，无论何时当一个Node unhealthy了，那么Node Controller会询问云服务提供商是否这个Node所在的VM任然可用，如果不可用，那么Node Controller会从自身维护的Node list中删除这个Node。

第三是监控Node的health状态。Node Controller负责在当Node不可达时更新Node Status中的Condition的Ready字段为Unknown（例如由于某些原因Node挂掉了，导致Node Controller收不到该Node的心跳信息），如果一段时间后Node还是unreachable，那么Node Controller会使用graceful termination来驱逐该Node上的所有Pods（默认设置ConditionUnknown的超时时间为40S，并且在5分钟后开始驱逐Pods）。Node Controller会检查每个Node的state每过`--node-monitor-period` 秒。



#### Heartbeats

Heartbeats, sent by Kubernetes nodes, help determine the availability of a node.

There are two forms of heartbeats: updates of `NodeStatus` and the [Lease object](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#lease-v1-coordination-k8s-io). Each Node has an associated Lease object in the `kube-node-lease` [namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces). Lease is a lightweight resource, which improves the performance of the node heartbeats as the cluster scales.

The kubelet is responsible for creating and updating the `NodeStatus` and a Lease object.

- The kubelet updates the `NodeStatus` either when there is change in status, or if there has been no update for a configured interval. The default interval for `NodeStatus` updates is 5 minutes (much longer than the 40 second default timeout for unreachable nodes).
- The kubelet creates and then updates its Lease object every 10 seconds (the default update interval). Lease updates occur independently from the `NodeStatus` updates. If the Lease update fails, the kubelet retries with exponential backoff starting at 200 milliseconds and capped at 7 seconds.

心跳，由Node发送，帮助决定Node的可用性。

有两种形式的心跳：NodeStatus更新，Lease Object。每个Node都会在 `kube-node-lease` [namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces)下关联一个Lease Object。Lease（租约）是一个轻量资源，它可以在集群缩放时候提高Node心跳的性能。

Node上的kubelet负责创建和更新NodeStatus以及Lease Object

- Kubelet负责在Node状态发生改变，或者在一个配置的周期后进行更新NodeStatus。默认的NodeStatus默认更新周期是5分钟（比默认的40S不可达Node超时时间长很多）
- Kubelet创建并更新自身的Lease Object每10S（默认更新周期）。Lease的更新跟NodeStatus的更新是相互独立的。如果Lease更新失败了，Kubelet会进行指数退避的重试机制，由200ms开始进行指数增长，上限为7S。



#### Reliability[ ](https://kubernetes.io/docs/concepts/architecture/nodes/#reliability)

In most cases, node controller limits the eviction rate to `--node-eviction-rate` (default 0.1) per second, meaning it won't evict pods from more than 1 node per 10 seconds.

在大多数情况下，Node Controller限制驱逐率`--node-eviction-rate`为0.1每秒（默认值），这意味着在10S内最多允许驱逐一个Node。

The node eviction behavior changes when a node in a given availability zone becomes unhealthy. The node controller checks what percentage of nodes in the zone are unhealthy (NodeReady condition is ConditionUnknown or ConditionFalse) at the same time. If the fraction of unhealthy nodes is at least `--unhealthy-zone-threshold` (default 0.55) then the eviction rate is reduced: if the cluster is small (i.e. has less than or equal to `--large-cluster-size-threshold` nodes - default 50) then evictions are stopped, otherwise the eviction rate is reduced to `--secondary-node-eviction-rate` (default 0.01) per second. The reason these policies are implemented per availability zone is because one availability zone might become partitioned from the master while the others remain connected. If your cluster does not span multiple cloud provider availability zones, then there is only one availability zone (the whole cluster).

Node的驱逐行为会改变，当Node所在的Zone变成了Unhealthy。Node Controller会检查同一时刻该Zone中的Node不健康比例（NodeReady Condition为Unknonw或者False）。如果该Zone中不健康Node比例大于 `--unhealthy-zone-threshold` (default 0.55) ，那么eviction rate会减少：如果集群规模比较小（i.e. 小于等于`--large-cluster-size-threshold`设置的值，默认50），那么驱逐行为会被停止，否则eviction rate会降低到`--secondary-node-eviction-rate` (default 0.01)每秒。为什么要针对Zone来实现这样的机制？这是因为一个可用的Zone可能因为网络分区问题跟master隔离开来了，然后其他Zone依然可用。如果你的集群没有分布在各个不同的云服务提供上上，那么你只有一个可用的Zone，即你的整个集群为一个Zone。

A key reason for spreading your nodes across availability zones is so that the workload can be shifted to healthy zones when one entire zone goes down. Therefore, if all nodes in a zone are unhealthy then the node controller evicts at the normal rate of `--node-eviction-rate`. The corner case is when all zones are completely unhealthy (i.e. there are no healthy nodes in the cluster). In such a case, the node controller assumes that there's some problem with master connectivity and stops all evictions until some connectivity is restored.

一个关键的原因为什么要分散你的Node到不同的可用Zone上是因为这样做的话，workload（工作负载）可以被转移到一个健康的Zones上，当某个Zone整体出现故障时。因此如果一个Zone中所有的Nodes都unhealthy了，那么Node Controller会以正常的速率`--node-eviction-rate`来驱逐他们。极端情况下，当所有的Zones都全部unhealthy了，那么Node Controller会认为是master的连接出现了问题了并停止所有的驱逐行为直到连接恢复。

The node controller is also responsible for evicting pods running on nodes with `NoExecute` taints, unless those pods tolerate that taint. The node controller also adds [taints](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) corresponding to node problems like node unreachable or not ready. This means that the scheduler won't place Pods onto unhealthy nodes.

Node Controller也负责驱逐Pods,如果该Pods所在的Node带有NoExecute taints，除非Pods本身容许这个taint。Node Contoller也会负责在 Node上新增与该Node问题相关的taints，比如Node不可达或者not ready。这就意味着scheduler不会把Pods放到这些Unhealthy的Nodes上。



#### Node capacity

Node objects track information about the Node's resource capacity (for example: the amount of memory available, and the number of CPUs). Nodes that [self register](https://kubernetes.io/docs/concepts/architecture/nodes/#self-registration-of-nodes) report their capacity during registration. If you [manually](https://kubernetes.io/docs/concepts/architecture/nodes/#manual-node-administration) add a Node, then you need to set the node's capacity information when you add it.

Node Objects追踪关于Node资源Capacity的信息（比如memory可用总量，CPU的数量）。自我注册的Nodes会在注册阶段报告它的capacity信息。如果你手动你添加了一个Node，那你需要在新增的时候设置capacity信息。

The Kubernetes [scheduler](https://kubernetes.io/docs/reference/generated/kube-scheduler/) ensures that there are enough resources for all the Pods on a Node. The scheduler checks that the sum of the requests of containers on the node is no greater than the node's capacity. That sum of requests includes all containers managed by the kubelet, but excludes any containers started directly by the container runtime, and also excludes any processes running outside of the kubelet's control.

Kubernetes的scheduler会确保Node上的所有Pods有足够的资源使用。scheduler会检查Node上container的总的请求数量是否没有超过node的capacity。总的请求量包括kubelet管理的所有容器，但是不包括用户直接用container runtime启动的容器，以及不由kebelet管理的进程。

> **Note:** If you want to explicitly reserve resources for non-Pod processes, see [reserve resources for system daemons](https://kubernetes.io/docs/tasks/administer-cluster/reserve-compute-resources/#system-reserved).
>
> 如果你想要明确保留资源给非Pod进程使用(即非kubernetes集群内的进程),查看 [reserve resources for system daemons](https://kubernetes.io/docs/tasks/administer-cluster/reserve-compute-resources/#system-reserved).

