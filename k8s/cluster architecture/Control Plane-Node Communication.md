```
provisioned: adj.预分配的
credentials: n. 资格;资历;资格证书;证明书;证件
instantiated: 实例化
subject to: 使遭受
bundle: 
```

[TOC]



### Control Plane-Node Communication

This document catalogs the communication paths between the control plane (really the apiserver) and the Kubernetes cluster. The intent is to allow users to customize their installation to harden the network configuration such that the cluster can be run on an untrusted network (or on fully public IPs on a cloud provider).

这篇文档介绍Control Plane（实际上是API Server）与kubernetes cluster的通信方式。这篇文章的意图是去允许用户进行定制化的安装来强化网络配置，比如使得集群可以在一个不可信的网络上运行。



#### Node to Control Plane 节点到控制面板

Kubernetes has a "hub-and-spoke" API pattern. All API usage from nodes (or the pods they run) terminate at the apiserver (none of the other control plane components are designed to expose remote services). The apiserver is configured to listen for remote connections on a secure HTTPS port (typically 443) with one or more forms of client [authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/) enabled. One or more forms of [authorization](https://kubernetes.io/docs/reference/access-authn-authz/authorization/) should be enabled, especially if [anonymous requests](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#anonymous-requests) or [service account tokens](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#service-account-tokens) are allowed.

Kubernetes拥有一个中心辐射的API模式。所有Nodes或者Pods使用的API都会结束在API Server上（所有其他的Control Plane组件都不是设计用于暴露远程服务的）。API Server在一个安全的HTTPS端口（通常是443）上配置了一种或多种客户端认证机制来监听远程连接。并且API Server需要开启一种或者多种授权机制来对客户端进行授权，尤其是在[anonymous requests](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#anonymous-requests) 或者 [service account tokens](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#service-account-tokens)被允许的时候。

Nodes should be provisioned with the public root certificate for the cluster such that they can connect securely to the apiserver along with valid client credentials. A good approach is that the client credentials provided to the kubelet are in the form of a client certificate. See [kubelet TLS bootstrapping](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/) for automated provisioning of kubelet client certificates.

Node需要预分配一个集群的公有的根证书，以此Node可以使用有效的客户端凭证来安全的连接到API Server上。一种好的处理方式是，通过客户端证书的方式来提供给Kubelet一个客户端凭证。查看[kubelet TLS bootstrapping](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/)来如何自动的给kubelet继续预分配客户端证书。

Pods that wish to connect to the apiserver can do so securely by leveraging a service account so that Kubernetes will automatically inject the public root certificate and a valid bearer token into the pod when it is instantiated. The `kubernetes` service (in all namespaces) is configured with a virtual IP address that is redirected (via kube-proxy) to the HTTPS endpoint on the apiserver.

想连接到API Server的Pods可以通过使用一个Service Account来进行安全的连接，以此Kubernetes可以在Pod实例化的时候自动注入公有根证书以及一个有效的bearer token（持有人凭证）到Pod中。

The control plane components also communicate with the cluster apiserver over the secure port.

As a result, the default operating mode for connections from the nodes and pods running on the nodes to the control plane is secured by default and can run over untrusted and/or public networks.

Control Plane的其他组件也通过secure port来跟API Server进行通信。

所以由于上述的功能，默认的（缺省的）连接操作模式，即从Nodes或者Pods建立的与Control Plane的连接是安全的，并且可以在任意不可信的或者共有的网络中运行。



#### Control Plane to node 控制面板到Node[ ](https://kubernetes.io/docs/concepts/architecture/control-plane-node-communication/#control-plane-to-node)

There are two primary communication paths from the control plane (apiserver) to the nodes. The first is from the apiserver to the kubelet process which runs on each node in the cluster. The second is from the apiserver to any node, pod, or service through the apiserver's proxy functionality.

Control Plane到Nodes主要有两种通信方式。第一种是API Server到kubelet进程（运行在集群中的每个Node中）。第二种是API Server到任何的Node，Pod或者Service通过API Server的Proxy功能。

##### apiserver to kubelet

The connections from the apiserver to the kubelet are used for:

- Fetching logs for pods.
- Attaching (through kubectl) to running pods.
- Providing the kubelet's port-forwarding functionality.

从API Server到Kubelet的连接一般用于：

- 从pods拉取日志
- 通过Kubelet连接上（挂到）运行中的Pods
- 提供Kubelet的端口转发功能

These connections terminate at the kubelet's HTTPS endpoint. By default, the apiserver does not verify the kubelet's serving certificate, which makes the connection subject to man-in-the-middle attacks, and **unsafe** to run over untrusted and/or public networks.

To verify this connection, use the `--kubelet-certificate-authority` flag to provide the apiserver with a root certificate bundle to use to verify the kubelet's serving certificate.

If that is not possible, use [SSH tunneling](https://kubernetes.io/docs/concepts/architecture/control-plane-node-communication/#ssh-tunnels) between the apiserver and kubelet if required to avoid connecting over an untrusted or public network.

Finally, [Kubelet authentication and/or authorization](https://kubernetes.io/docs/admin/kubelet-authentication-authorization/) should be enabled to secure the kubelet API.

这些连接终止与kubelet的https端口。默认情况下，API Server不会去校验Kubelet的服务证书，这可能会导致连接遭受到中间人攻击，因此在不受信的网络或者公网环境下是不安全的。

如果要验证这个连接，可以使用`--kubelet-certificate-authority`参数以提供给API Server一个根证书包（bundle)，通过该根证书包来来验证Kubelet的服务证书。

如果无法实现上述方案，可以在API Server与Kebelet之间建立SSH通道来确保通信安全，以此避免在不受信或者公网环境下的进行连接。

最后，Kubelet应该启用认证、授权来保护Kubelet的API。



##### apiserver to nodes, pods, and services

The connections from the apiserver to a node, pod, or service default to plain HTTP connections and are therefore neither authenticated nor encrypted. They can be run over a secure HTTPS connection by prefixing `https:` to the node, pod, or service name in the API URL, but they will not validate the certificate provided by the HTTPS endpoint nor provide client credentials so while the connection will be encrypted, it will not provide any guarantees of integrity. These connections **are not currently safe** to run over untrusted and/or public networks.

API Server到Node，Pod或者Service的默认连接为Http连接，因此没有任何的认证或者加密。也可以通过把连接前缀改为`https`来建立安全的HTTPS连接，但是不会验证Node，Pod或者Service提供的HTTPS证书，也不会提供API Server自己的客户端证书给对方，所以连接只会提供HTTPS的加密功能，但是不会提供任何其他的HTTPS的功能保证。这些连接目前不能安全的运行在不受信或者公有网络上。



##### SSH tunnels

Kubernetes supports SSH tunnels to protect the control plane to nodes communication paths. In this configuration, the apiserver initiates an SSH tunnel to each node in the cluster (connecting to the ssh server listening on port 22) and passes all traffic destined for a kubelet, node, pod, or service through the tunnel. This tunnel ensures that the traffic is not exposed outside of the network in which the nodes are running.

SSH tunnels are currently deprecated so you shouldn't opt to use them unless you know what you are doing. The Konnectivity service is a replacement for this communication channel.

Kubernetes支持SSH通道来保护Control Plane到Node的通信通道。在这种配置下面，API Server会初始化到集群中各个Node的SSH通道（连接到SSH服务的22端口上），并通过该通道传输所有的流量到Kubelet，Node，Pod以及Service。SSH通道确保流量不会暴露到Node所在的网络外面。

SSH Tunnel目前已经被废弃了，所有你不应该去操作它，除非你知道自己在做什么。Konnectivity服务是SSH Tunnel的一个替代品。



##### Konnectivity service

**FEATURE STATE:** `Kubernetes v1.18 [beta]`  该feature的状态：Kubernetes v1.18[beta]

As a replacement to the SSH tunnels, the Konnectivity service provides TCP level proxy for the control plane to cluster communication. The Konnectivity service consists of two parts: the Konnectivity server and the Konnectivity agents, running in the control plane network and the nodes network respectively. The Konnectivity agents initiate connections to the Konnectivity server and maintain the network connections. After enabling the Konnectivity service, all control plane to nodes traffic goes through these connections.

Follow the [Konnectivity service task](https://kubernetes.io/docs/tasks/extend-kubernetes/setup-konnectivity/) to set up the Konnectivity service in your cluster.

作为SSH Tunnel的替代品，Konnectivity服务提供TCP级别的代理来满足Control Plane到集群的通信需求。Konnectivity服务由两部分组成：Konnectivity Server端以及Agents端，相应的运行在Control Plane网络以及Node网络中。Agent初始化连接到Server端并维护这个网络连接。开启Konnectivity服务后,所有Control Plane到Nodes的流量会通过该连接通道进行传输通信。

参考 [Konnectivity service task](https://kubernetes.io/docs/tasks/extend-kubernetes/setup-konnectivity/)来在你的集群中设置Konnectivity Service。



