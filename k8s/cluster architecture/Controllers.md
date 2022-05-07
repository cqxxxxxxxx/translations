```
robotics: n.机器人学，机器人的复数
automation: n.自动化，自动操作
thermostat: n.恒温器，自动调温器
regulate: v.控制，调节，管理
carry out: 执行，实施
versus: prep. 对，对抗；与……相对，与……相比
```





### Controllers

In robotics and automation, a *control loop* is a non-terminating loop that regulates the state of a system.

Here is one example of a control loop: a thermostat in a room.

When you set the temperature, that's telling the thermostat about your *desired state*. The actual room temperature is the *current state*. The thermostat acts to bring the current state closer to the desired state, by turning equipment on or off.

In Kubernetes, controllers are control loops that watch the state of your [cluster](https://kubernetes.io/docs/reference/glossary/?all=true#term-cluster), then make or request changes where needed. Each controller tries to move the current cluster state closer to the desired state.

在机器人学与自动化领域，控制回路是一个不会终止的循环用于管理系统的状态。

这有一个控制回路的例子：房间里的恒温器。

当你设置了温度，这就是告诉了恒温器一个你期望的状态。房间当前的温度就是当前的状态。恒温器会通过开关设备来把当前的状态（温度）调整到目标期望的状态（温度）。

在Kubernetes中，Controllers就是控制回路，它会观察你集群的状态，然后在必要的时候做或者进行请求来进行一些改变。每一个Controller都尝试调整集群的当前状态来更加接近期望的状态。



#### Controller pattern 控制器模式

A controller tracks at least one Kubernetes resource type. These [objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/#kubernetes-objects) have a spec field that represents the desired state. The controller(s) for that resource are responsible for making the current state come closer to that desired state.

The controller might carry the action out itself; more commonly, in Kubernetes, a controller will send messages to the [API server](https://kubernetes.io/docs/concepts/overview/components/#kube-apiserver) that have useful side effects. You'll see examples of this below.

一个控制器至少追踪一种Kubernetes的资源类型。这些Objects有一个特定的字段表名其期望状态。该资源的控制器负责将当前状态调整为更加接近期望的状态。

Controller也许会自己执行动作来促使当前状态更加接近期望状态。但是在Kubernetes中更加常见的是，Controller发送消息给API Server来对状态进行一些有效的侧面的间接的影响。以下是一些例子（直接控制，通过API Server间接控制）。

##### Control via API server 通过API Server进行控制[ ](https://kubernetes.io/docs/concepts/architecture/controller/#control-via-api-server)

The [Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/) controller is an example of a Kubernetes built-in controller. Built-in controllers manage state by interacting with the cluster API server.

Job is a Kubernetes resource that runs a [Pod](https://kubernetes.io/docs/concepts/workloads/pods/), or perhaps several Pods, to carry out a task and then stop.

(Once [scheduled](https://kubernetes.io/docs/concepts/scheduling-eviction/), Pod objects become part of the desired state for a kubelet).

Job Controller是Kubernetes内置Controller的一个例子。内置的Controllers通过跟集群的 API Server交互来进行状态管理。

Job是一种Kubernetes资源,可以运行一个或者多个Pod，来执行一个任务并终止它。

（一旦被调度了，Pod Obeject就变成kubelet的一部分状态）

When the Job controller sees a new task it makes sure that, somewhere in your cluster, the kubelets on a set of Nodes are running the right number of Pods to get the work done. The Job controller does not run any Pods or containers itself. Instead, the Job controller tells the API server to create or remove Pods. Other components in the [control plane](https://kubernetes.io/docs/reference/glossary/?all=true#term-control-plane) act on the new information (there are new Pods to schedule and run), and eventually the work is done.

当Job Controller看到一个新的Task，它会确保，在你集群的某个地方，在一组Nodes上的Kubelets正在运行着正确数量的Pods来完成预期的工作。Job Controller它自身不会运行Pods或者Containers，相反的Job Controller告诉API Server去创建和删除Pods。其他在Control Plane中的组件在收到新信息后会做出相应响应（对这些新的Pods进行调度并运行），最终保证工作顺利完成。

After you create a new Job, the desired state is for that Job to be completed. The Job controller makes the current state for that Job be nearer to your desired state: creating Pods that do the work you wanted for that Job, so that the Job is closer to completion.

在你创建一个新的Job后，Job所期望的状态就是把负责的工作完成。Job Controller负责将Job当前状态不断调整接近期望状态：创建Pods来执行你的工作，以此Job会接近完成。

Controllers also update the objects that configure them. For example: once the work is done for a Job, the Job controller updates that Job object to mark it `Finished`.

(This is a bit like how some thermostats turn a light off to indicate that your room is now at the temperature you set).

Controllers也会更新Objects的配置项。比如，一旦Job的工作完成，Job Controller会更新Job Object标记成完成。

（这就像恒温器把灯关掉来标识你房间的温度已经达到了你期望的温度）



##### Direct control 直接控制

By contrast with Job, some controllers need to make changes to things outside of your cluster.

For example, if you use a control loop to make sure there are enough [Nodes](https://kubernetes.io/docs/concepts/architecture/nodes/) in your cluster, then that controller needs something outside the current cluster to set up new Nodes when needed.

Controllers that interact with external state find their desired state from the API server, then communicate directly with an external system to bring the current state closer in line.

(There actually is a controller that horizontally scales the nodes in your cluster. See [Cluster autoscaling](https://kubernetes.io/docs/tasks/administer-cluster/cluster-management/#cluster-autoscaling)).

与Job Controller相比，一些Controller需要修改集群外部的一些事情。

比如，如果你使用控制回路来确保你集群中有足够的Node数量，那么Controller就需要一些集群外部的支持来在合适的时候创建新的Node。

跟外部状态交互的Controllers先通过API Server发现他们的期望状态，然后跟外部系统直接交互来控制当前状态不断接近目标期望状态。

（实际上有一个控制可以在集群中对Nodes进行水平扩展，参考 [Cluster autoscaling](https://kubernetes.io/docs/tasks/administer-cluster/cluster-management/#cluster-autoscaling)）



#### Desired versus current state 期望状态VS当前状态

Kubernetes takes a cloud-native view of systems, and is able to handle constant change.

Your cluster could be changing at any point as work happens and control loops automatically fix failures. This means that, potentially, your cluster never reaches a stable state.

As long as the controllers for your cluster are running and able to make useful changes, it doesn't matter if the overall state is or is not stable.

Kubernetes采用云原生视图来观察系统，并且可以处理常量变化。？？？

你的集群可能在任何时间点发送变化，只要发生了work，并且控制回路会自动修复故障。这意味着，潜在的可能就是你的集群永远达不到一个稳定的状态。

只要集群的Controllers在运行并且可以进行有效的修改，那么集群整体状态的稳定与否就不是一个问题。



#### Design 设计[ ](https://kubernetes.io/docs/concepts/architecture/controller/#design)

As a tenet of its design, Kubernetes uses lots of controllers that each manage a particular aspect of cluster state. Most commonly, a particular control loop (controller) uses one kind of resource as its desired state, and has a different kind of resource that it manages to make that desired state happen. For example, a controller for Jobs tracks Job objects (to discover new work) and Pod objects (to run the Jobs, and then to see when the work is finished). In this case something else creates the Jobs, whereas the Job controller creates Pods.

It's useful to have simple controllers rather than one, monolithic set of control loops that are interlinked. Controllers can fail, so Kubernetes is designed to allow for that.

Kubernetes设计的其中一个原则，使用许多Controllers，各自管理集群特定某个方面的状态。更常见的是，一个特定的控制回路（Controller）把一种资源作为它的期望状态，并通过管理其他类型的资源来达到期望状态。比如说，一个Job Controller追踪Job Object（用于发现新的work）和Pod Object（用于运行Jobs，并观察什么时候work完成）。在这个场景下，其他东西创建Jobs，然而Job Controller负责创建Pods。

这是很有用的，有一个简单的Controller，而不是一整块互相关联的控制回路。Controllers可能会失败出现问题，而Kubernetes也是设计成容许出现这种情况。

> **Note:**
>
> There can be several controllers that create or update the same kind of object. Behind the scenes, Kubernetes controllers make sure that they only pay attention to the resources linked to their controlling resource.
>
> For example, you can have Deployments and Jobs; these both create Pods. The Job controller does not delete the Pods that your Deployment created, because there is information ([labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels)) the controllers can use to tell those Pods apart.
>
> 可以有不同的Controllers来创建更新同一种类型的Object。在这种情况下，Kubernetes Controllers会确保他们只会关心自己关联的控制资源。
>
> 比如你可以有Deployment和Job Controller；他们都会创建Pods。Job Controller 不会删除Deployment Controller创建的Pods，因为Controller可以通过信心（标签）来区分出Pods。



#### Ways of running controllers 运行Controller的方式[ ](https://kubernetes.io/docs/concepts/architecture/controller/#running-controllers)

Kubernetes comes with a set of built-in controllers that run inside the [kube-controller-manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/). These built-in controllers provide important core behaviors.

The Deployment controller and Job controller are examples of controllers that come as part of Kubernetes itself ("built-in" controllers). Kubernetes lets you run a resilient control plane, so that if any of the built-in controllers were to fail, another part of the control plane will take over the work.

You can find controllers that run outside the control plane, to extend Kubernetes. Or, if you want, you can write a new controller yourself. You can run your own controller as a set of Pods, or externally to Kubernetes. What fits best will depend on what that particular controller does.

Kubernetets带有一系列内置的Controller，他们运行在 [kube-controller-manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)中。这些内置的Controllers提供着重要的核心的行为能力。

Deployment Controller和Job Controller是Kubernetes内置Controller的一个例子。Kubernetes运行着一个弹性的Control Plane,所以如果有任何的内置Controller失败故障了，Control Plane的其他部分会接替他们的工作。

你可以发现Controller运行在Control Plane外部，这些Controller用于扩展Kubernetes。或者只要你想，你可以自己写一个新的Controller。你可以运行你的Controller作为一组Pods，或者运行在Kubernetes集群外部。至于怎么做那取决于你Controller具体的需要实现的功能。



