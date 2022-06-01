

# Kuberneters Tutorial

```markdown
objectives: 目的
address: 地址，设法解决
failure: 失败，故障
Deployment: 部署， n.部署图(在k8s中的意义)
replica: 复制，副本
Objects: K8S中可以理解为概念模型？K8S对象？
ctl: ConTroL的缩写
```

## 官网Tutorial学习记录

链接：[https://kubernetes.io/docs/tutorials/](https://kubernetes.io/docs/tutorials/)
词汇表：[https://kubernetes.io/zh/docs/reference/glossary/?core-object=true](https://kubernetes.io/zh/docs/reference/glossary/?core-object=true)

## 一. Learn Kubernetes Basics(学习k8s基础知识)

### Create a Cluster

#### Using Minikube to Create a Cluster

Objectives:

1. Learn what a Kuberneters cluster is.

```markdown
Kubernetes coordinates a highly available cluster of computers that are connected 
to work as a single unit.
k8s协调一个高可用的计算机集群，that 互相连接作为一个单元去工作

Kubernetes automates the distribution and scheduling of application containers acro
ss a cluster in a more efficient way.
k8s以一种更有效率的方式在cluster中自动的分发和调度应用容器

A kubernetes Cluster consists of two types of resources:
The Master Which manage the cluster.(Master节点，协调调度整个集群)
The Nodes Which host the running applications.(Node节点，运行应用容器)
```

![Kubernetes cluster](https://d33wubrfki0l68.cloudfront.net/99d9808dcbf2880a996ed50d308a186b5900cec9/40b94/docs/tutorials/kubernetes-basics/public/images/module_01_cluster.svg)
【K8S集群】

```markdown
The Master is responsible for managing the cluster. 
主节点主要负责管理整个集群

The master coordinates all activities in your cluster, such as scheduling applicati
ons, maintaining applications' desired state, scaling applications, and rolling out
new updates.
主节点负责协调集群的所有活动，比如调度应用，维护应用期望的状态，自动伸缩应用，动态更新应用。

---

A node is a VM or a physical computer that serves as a worker machine in a Kubernet
es cluster.
Node节点可以是一个虚拟机或者物理机，他在整个K8S集群中作为一个工作节点存在。

Each node has a Kubelet, which is an agent for managing the node and communicating 
with the Kubernetes master. The node should also have tools for handling container 
operations, such as Docker or rkt. 
每个Work Node都有一个Kubelet(作为一个代理，用于管理当前节点以及和master节点沟通)。此外每个节点还
需要有比如Docker或者rkt作为操作管理容器的工具。

When you deploy applications on Kubernetes, you tell the master to start the applic
ation containers. The master schedules the containers to run on the cluster's nodes.
The nodes communicate with the master using the Kubernetes API, which the master ex
poses. End users can also use the Kubernetes API directly to interact with the clus
ter.
当你在K8S上部署一个应用时，你通过Master去启动一个应用容器。Master则会自动调度一个node去运行这个应用。
集群node通过Kubernetes API跟master节点通信。因此用户也可以直接使用API去跟集群交互，从而跳过Master
节点。

```

2. Learn what Minikube is.

```markdown
Minikube is a lightweight Kubernetes implementation that creates a VM on your local
machine and deploys a simple cluster containing only one node. 
Minikube是一个轻量级的k8s实现，他可以在你本地机器上创建一个VM并在上面部署一个单节点的K8S集群

--相关命令
##启动minikube
./minikube start --image-mirror-country='cn' --registry-mirror=https://registry.docker-cn.com --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
```

3. Start a Kuberneters cluster using an online terminal

### Deploy an App
#### Using Kubectl to Create a Deployment 
Objectives:

1. Learning about application Deloyment.

```markdown
A Deployment is responsible for creating or updating instances of your application.
Deloyment主要负责应用实例的创建和升级。

The Deployment instructs Kubernetes how to create and update instances of your application.
Deloyment指示K8S如何创建和更新你的应用实例。

Once the application instances are created, a Kubernetes Deployment Controller 
continuously monitors those instances. If the Node hosting an instance goes down or
is deleted, the Deployment controller replaces the instance with an instance on 
another Node in the cluster. This provides a self-healing mechanism to address 
machine failure or maintenance.
一旦应用实例被创建了，K8S的Deloyment Controller会持续监控这些实例。 如果所在Node挂掉或者被删除了，
Deloyment Controller会在集群的其他节点上创建一个实例替换掉挂掉的instance。这样就提供了一个自我恢复
的机制，以此解决机器故障和维持服务稳定的问题。

```

![](https://d33wubrfki0l68.cloudfront.net/152c845f25df8e69dd24dd7b0836a289747e258a/4a1d2/docs/tutorials/kubernetes-basics/public/images/module_02_first_app.svg)
【K8S集群】


2. Deloy your first app on Kubernates with kubectl.

```markdown
Kubectl 命令格式 => kubectl action resource
kubectl get nodes/deloyments/pods ...
## kubectl create deployment [name] --image=[image url] => 这个命令在1.16后好像不行了，apiversion更新了
kubectl run [NAME] --image=[IMAGE] => 运行一个容器
```

### Explore Your App
#### Viewing Pods and Nodes
objectives:

1. Learn about Kubernetes Pods.

```markdown
A Pod is a group of one or more application containers (such as Docker or rkt) and 
includes shared storage (volumes), IP address and information about how to run them.
Pod是指一组单个或者多个的应用容器，包括其中的共享存储，IP地址和如何运行应用的信息，比如容器使用的镜像版本，
使用的port等等。

A Pod models an application-specific "logical host" and can contain different appli-
cation containers which are relatively tightly coupled.
Pod提供一个模型，用来描述一个运行特定应用的逻辑主机。Pod可以包含不同的应用容器，而不限于单个容器，只要
他们是紧密耦合的。

Pods are the atomic unit on the Kubernetes platform. 
Pod是K8S平台上的最小的原子单元。
```
![](https://d33wubrfki0l68.cloudfront.net/fe03f68d8ede9815184852ca2a4fd30325e5d15a/98064/docs/tutorials/kubernetes-basics/public/images/module_03_pods.svg)
【Pods 概览】

2. Learn about kuberneters Nodes.

```markdown
A Node is a worker machine in Kubernetes and may be either a virtual or a physical 
machine, depending on the cluster.
Node在K8S中作为工作节点，可以是物理或者虚拟计算机。

Node can have multiple pods, and the Kubernetes master automatically handles sche-
duling the pods across the Nodes in the cluster. 
Node中可以运行多个Pod，K8S的Master节点会自动调度Pod到集群中不同的Node中。

Every Kubernetes Node runs at least:
1.Kubelet, a process responsible for communication between the Kubernetes Master 
and the Node; it manages the Pods and the containers running on a machine.
Kubelet，一个进程负责Master和当前Node的通信，以及管理当前Node中的Pods和运行的容器。

2.A container runtime (like Docker, rkt) responsible for pulling the container image 
from a registry, unpacking the container, and running the application.
容器引擎(Docker，rkt)，负责从仓库拉取容器镜像并运行应用。
```
![](https://d33wubrfki0l68.cloudfront.net/5cb72d407cbe2755e581b6de757e0d81760d5b86/a9df9/docs/tutorials/kubernetes-basics/public/images/module_03_nodes.svg)
                                              

3. Troubleshoot deloyed applications.

```markdown
kubectl get - list resources => 查看资源
kubectl describe - show detailed information about a resource => 查资源详情
kubectl logs - print the logs from a container in a pod => 打印指定Pod的指定容器的日志
kubectl exec - execute a command on a container in a pod => 在指定Pod的指定容器中执行命令

kubectl exec -ti $POD_NAME bash => 在进入指定pod(如果只有一个容器则不用指定)开启一个bash，类似docker的命令
```


### Expose Your App Publicly
#### Using a Service to Expose Your App
Objectives:

1. Learn about Service in Kubernetes.

```markdown
A Kubernetes Service is an abstraction layer which defines a logical set of Pods 
and enables external traffic exposure, load balancing and service discovery for 
those Pods.
K8S的Service是一个抽象层，它定义了一个Pods的逻辑分组，并为这些Pod提供外部流量暴露，负载均衡，服务发
现的能力。

A Service routes traffic across a set of Pods. Services are the abstraction that 
allow pods to die and replicate in Kubernetes without impacting your application. 
Discovery and routing among dependent Pods (such as the frontend and backend compon-
ents in an application) is handled by Kubernetes Services.
Service把流量路由到一组Pods中。Service允许其中的Pod死亡或者复制再生而不会影响你的应用。Service处
理Pod的发现和路由。比如应用A依赖应用B，那么A如何发现并连接到B呢，Service就提供了这个功能。

为什么需要Service？单使用Pod带来的问题。 
Pod是有生命周期的，当一个WorkNode挂掉，Node中的Pod也会全部丢失。ReplicaSet会动态的驱动整个集群
回归desired期望的状态，通过在其他Node上创建新的Pod来恢复你的应用。
每个Pod在K8S集群中都有唯一的IP(即使在同一个Node中)，所以当Pod重新创建或者迁移时需要有一种机制来
自动协调Pod的IP变化等对你应用造成影响。

Although each Pod has a unique IP address, those IPs are not exposed outside the 
cluster without a Service. Services allow your applications to receive traffic. 
Services can be exposed in different ways by specifying a type in the ServiceSpec:
虽然每个Pod都有唯一的IP地址，但是这些IP不会暴露到K8S集群外部（在没有使用Service的情况下）。通过使用
Service你的应用可以接收到外部流量即对外暴露。ServiceSpec可以配置具体的暴露方式,选项如下。
1.ClusterIP (default) - Exposes the Service on an internal IP in the cluster. This 
type makes the Service only reachable from within the cluster. 
ClusterIP(默认):通过一个集群内部IP暴露Service，因此该Service只能在集群内部被访问。
2.NodePort - Exposes the Service on the same port of each selected Node in the 
cluster using NAT. Makes a Service accessible from outside the cluster using 
<NodeIP>:<NodePort>. Superset of ClusterIP.
NodePort:通过NAT来暴露所在Node的跟Service相同的端口，因此Service可以被K8S集群外所访问到，通过
NodeIP:NodePort方式。 类似docker的host网络模式。
3.LoadBalancer - Creates an external load balancer in the current cloud (if supported)
and assigns a fixed, external IP to the Service. Superset of NodePort.
LoadBalancer:通过创建一个外部的LB。是NodePort方式的超集。云服务商提供的？
4.ExternalName - Exposes the Service using an arbitrary name (specified by externalNa-
me in the spec) by returning a CNAME record with the name. No proxy is used. This 
type requires v1.7 or higher of kube-dns.
ExternalName: 使用任意的name来暴露Service，这个name会返回一个CNAME记录，CNAME会指向具体的Ser-
vice。只有v1.7或更高版本的kube-dns支持。
```
![module_04_services.svg](https://d33wubrfki0l68.cloudfront.net/cc38b0f3c0fd94e66495e3a4198f2096cdecd3d5/ace10/docs/tutorials/kubernetes-basics/public/images/module_04_services.svg)![module_04_labels.svg](https://d33wubrfki0l68.cloudfront.net/b964c59cdc1979dd4e1904c25f43745564ef6bee/f3351/docs/tutorials/kubernetes-basics/public/images/module_04_labels.svg)
Understand how labels and LabelSelector objects relate to a Service.

```markdown
Services match a set of Pods using labels and selectors, a grouping primitive that 
allows logical operation on objects in Kubernetes. Labels are key/value pairs atta-
ched to objects and can be used in any number of ways:
1.Designate objects for development, test, and production
2.Embed version tags
3.Classify an object using tags
Service通过使用Label和LabelSelector来匹配管理一组Pods。Label是KV结构附属在Objects上，通常被用
于区分不同环境对象、版本号、tag归类


--相关命令
kubectl expose rc tom --type=NodePort --port=8080 # 创建一个服务
kubectl label rc tom owner=cqxxxxxxxx  # 添加一个label
```

3. Expose an application outside of a Kubernetes cluster using a Service.

### Scale Your App
#### Running Mutiple Instances of Your App
Objectives:

1. Scale an app using kubectl.

```markdown
Scaling is accomplished by changing the number of replicas in a Deployment
自动水平扩容或者缩小通过改变Deloyment的复制/副本数量来实现。

Scaling out a Deployment will ensure new Pods are created and scheduled to Nodes 
with available resources. 
水平扩容Deloyment在创建和调度Pod的时候会确保Node有足够的资源运行该Pod。

Scaling will increase the number of Pods to the new desired state. 
Scaling会增加Pod的数量到预期值。

Scaling to zero is also possible, and it will terminate all Pods of the specified 
Deployment.
Scaling到0也是可能的，它会终止
```
![module_05_scaling1.svg](https://cdn.nlark.com/yuque/0/2020/svg/701303/1578388653186-0c919bd8-58ee-4e15-a8c4-84971719c03b.svg#align=left&display=inline&height=291&name=module_05_scaling1.svg&originHeight=150&originWidth=165&size=23261&status=done&style=none&width=320)          ![module_05_scaling2.svg](https://cdn.nlark.com/yuque/0/2020/svg/701303/1578388646399-a05a34c6-cdf5-473d-8e80-3a743ebdce24.svg#align=left&display=inline&height=291&name=module_05_scaling2.svg&originHeight=150&originWidth=165&size=29933&status=done&style=none&width=320)
                  【水平扩容前】                          =>                           【后】

```markdown
Running multiple instances of an application will require a way to distribute the 
traffic to all of them. Services have an integrated load-balancer that will distri-
bute network traffic to all Pods of an exposed Deployment. Services will monitor 
continuously the running Pods using endpoints, to ensure the traffic is sent only 
to available Pods.
运行一个应用的多个实例时需要分配流量到所有的实例上。Service集成了LB来分配网络流量到所有暴露的Pod.
Service会持续不断的通过endpoint来监控运行的Pod，确保流量分配到可用的Pod上。

--相关命令
kubectl scale --replicas=4 rc/tom 设置4个replica
```

### Update Your App
#### Performing a Rolling Update
Objectives:

1. Perform a rolling update using kubectl.

```markdown
Rolling updates allow Deployments' update to take place with zero downtime by incre-
mentally updating Pods instances with new ones. 
滚动升级允许Deployment不停机更新，通过递增的更新Pod实例到更新后的版本。

In Kubernetes, updates are versioned and any Deployment update can be reverted to 
previous (stable) version.
K8S中，每个更新都会标记一个版本号，任意的Deloyment更新都可以通过版本号回滚到先前的版本。

If a Deployment is exposed publicly, the Service will load-balance the traffic only
to available Pods during the update.

通过Rolling updates我们可以方便的实现以下需求:
1.Promote an application from one environment to another (via container image updates)
2.Rollback to previous versions
3.Continuous Integration and Continuous Delivery of applications with zero downtime

--相关命令
 #Set a deployment's nginx container image to 'nginx:1.9.1', and its busybox 
 #container image to 'busybox'.
 kubectl set image deployment/nginx busybox=busybox nginx=nginx:1.9.1
 
 #Rollback to the previous deployment
 kubectl rollout undo deployment/abc
```
![module_06_rollingupdates1.svg](https://cdn.nlark.com/yuque/0/2020/svg/701303/1578396890302-519b6d09-9019-402b-ac60-b7237ce27351.svg#align=left&display=inline&height=150&name=module_06_rollingupdates1.svg&originHeight=150&originWidth=165&size=29933&status=done&style=none&width=165)![module_06_rollingupdates2.svg](https://cdn.nlark.com/yuque/0/2020/svg/701303/1578396896186-2f416090-b9bc-4a7f-9df7-42c0002fa121.svg#align=left&display=inline&height=150&name=module_06_rollingupdates2.svg&originHeight=150&originWidth=165&size=44562&status=done&style=none&width=165)![module_06_rollingupdates3.svg](https://cdn.nlark.com/yuque/0/2020/svg/701303/1578396900853-1d55eeeb-9fdd-4f3f-b1c1-93ebc1d3f4b0.svg#align=left&display=inline&height=150&name=module_06_rollingupdates3.svg&originHeight=150&originWidth=165&size=39437&status=done&style=none&width=165)![module_06_rollingupdates4.svg](https://cdn.nlark.com/yuque/0/2020/svg/701303/1578396905059-353a3580-73ce-4fcd-8715-54ff0c7ded8c.svg#align=left&display=inline&height=150&name=module_06_rollingupdates4.svg&originHeight=150&originWidth=165&size=41004&status=done&style=none&width=165)

【滚动更新的流程图】










# 二. Configuring Redis using a ConfigMap

Objectives:

1. Create a`kustomization.yaml`file containing:
   - a ConfigMap generator
   - a Pod resource config using the ConfigMap
2. Apply the directory by running `kubectl apply -k ./`
2. Verify that the configuration was correctly applied.


