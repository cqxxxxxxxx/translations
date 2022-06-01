```
Prerequisites: 预备知识
```



### Pod Topology Spread Constraints 拓扑扩展约束

You can use *topology spread constraints* to control how [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) are spread across your cluster among failure-domains such as regions, zones, nodes, and other user-defined topology domains. This can help to achieve high availability as well as efficient resource utilization.

你可以使用拓扑扩展约束来控制如何在你的集群故障域（例如地区，区域，节点和其他用户自定义拓扑域）中的分布。这可以帮助你实现高可用以及资源有效利用。





#### Prerequisites 前置知识

##### Node Labels Node的标签

Topology spread constraints rely on node labels to identify the topology domain(s) that each Node is in. For example, a Node might have labels: `node=node1,zone=us-east-1a,region=us-east-1`

拓扑分布约束依赖于Node的标签来定位每个Node所在的拓扑域。比如说一个Node也许有一些标签: `node=node1,zone=us-east-1a,region=us-east-1`

Suppose you have a 4-node cluster with the following labels:  

假设你有一个4个节点的集群,这些节点带有如下的标签

```
NAME    STATUS   ROLES    AGE     VERSION   LABELS
node1   Ready    <none>   4m26s   v1.16.0   node=node1,zone=zoneA
node2   Ready    <none>   3m58s   v1.16.0   node=node2,zone=zoneA
node3   Ready    <none>   3m17s   v1.16.0   node=node3,zone=zoneB
node4   Ready    <none>   2m43s   v1.16.0   node=node4,zone=zoneB
```

Then the cluster is logically viewed as below:

那么这个集群逻辑视图就如下：

![image-20201006204420931](../../../../../assets/image-20201006204420931.png)

Instead of manually applying labels, you can also reuse the [well-known labels](https://kubernetes.io/docs/reference/kubernetes-api/labels-annotations-taints/) that are created and populated automatically on most clusters.

而不是手动创建标签，你也可以重用[well-known labels](https://kubernetes.io/docs/reference/kubernetes-api/labels-annotations-taints/)，这些标签在大多数集群上都是自动创建和激活的。

