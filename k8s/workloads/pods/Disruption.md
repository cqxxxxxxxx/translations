```
disruption: n.中断;扰乱;混乱
```

[TOC]



# Disruptions 干扰

This guide is for application owners who want to build highly available applications, and thus need to understand what types of disruptions can happen to Pods.

It is also for cluster administrators who want to perform automated cluster actions, like upgrading and autoscaling clusters.

这个指南是给那些想要构建高可用应用的作者的，因此需要知道哪些可能的干扰可能会在Pod上发生。

这个指南也是为了给集群管理者，who想要执行自动化的集群操作，比如升级和自动扩展集群。



## Voluntary and involuntary disruptions  自愿和非自愿干扰

Pods do not disappear until someone (a person or a controller) destroys them, or there is an unavoidable hardware or system software error.

Pod不会消失，直到有人或者controller销毁他们，或者一些不可阻止的硬件或者系统软件错误发生了。

We call these unavoidable cases *involuntary disruptions* to an application. Examples are:

- a hardware failure of the physical machine backing the node
- cluster administrator deletes VM (instance) by mistake
- cloud provider or hypervisor failure makes VM disappear
- a kernel panic
- the node disappears from the cluster due to cluster network partition
- eviction of a pod due to the node being [out-of-resources](https://kubernetes.io/docs/tasks/administer-cluster/out-of-resource/).

Except for the out-of-resources condition, all these conditions should be familiar to most users; they are not specific to Kubernetes.

We call other cases *voluntary disruptions*. These include both actions initiated by the application owner and those initiated by a Cluster Administrator. Typical application owner actions include:

- deleting the deployment or other controller that manages the pod
- updating a deployment's pod template causing a restart
- directly deleting a pod (e.g. by accident)

Cluster administrator actions include:

- [Draining a node](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/) for repair or upgrade.
- Draining a node from a cluster to scale the cluster down (learn about [Cluster Autoscaling](https://kubernetes.io/docs/tasks/administer-cluster/cluster-management/#cluster-autoscaler) ).
- Removing a pod from a node to permit something else to fit on that node.

These actions might be taken directly by the cluster administrator, or by automation run by the cluster administrator, or by your cluster hosting provider.

Ask your cluster administrator or consult your cloud provider or distribution documentation to determine if any sources of voluntary disruptions are enabled for your cluster. If none are enabled, you can skip creating Pod Disruption Budgets.

> **Caution:** Not all voluntary disruptions are constrained by Pod Disruption Budgets. For example, deleting deployments or pods bypasses Pod Disruption Budgets.