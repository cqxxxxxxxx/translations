```
likewise: 同样的
identical： adj.完全相同的，一样的
comprehensive： adj.综合的，全面的
disruptive： adj. 破坏性的
```

[TOC]



### Pod Lifecycle

This page describes the lifecycle of a Pod. Pods follow a defined lifecycle, starting in the `Pending` [phase](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase), moving through `Running` if at least one of its primary containers starts OK, and then through either the `Succeeded` or `Failed` phases depending on whether any container in the Pod terminated in failure.

Whilst a Pod is running, the kubelet is able to restart containers to handle some kind of faults. Within a Pod, Kubernetes tracks different container [states](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-states) and handles

In the Kubernetes API, Pods have both a specification and an actual status. The status for a Pod object consists of a set of [Pod conditions](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-conditions). You can also inject [custom readiness information](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-readiness-gate) into the condition data for a Pod, if that is useful to your application.

Pods are only [scheduled](https://kubernetes.io/docs/concepts/scheduling-eviction/) once in their lifetime. Once a Pod is scheduled (assigned) to a Node, the Pod runs on that Node until it stops or is [terminated](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination).

这一页描述Pod的生命周期。Pods遵循一个特定的生命周期，以Pending阶段开始，转向Running阶段，如果有至少一个主要容器启动成功的话，然后是Successed或者Failed阶段，这取决于Pod中是否有容器以失败状态退出。

Pod运行时，kubelet可以在容器出现故障时重启容器。在Pod内部，Kubernetes追踪不同容器的状态并进行处理。

在Kubernetes API中，Pods有着它的规范和状态。Pod Object的状态由一组Pod Conditions组成。你也可以注入自定义的准备就绪的状态到Pod的condition data中，如果你应用需要的话。

Pods在他们生命周期中只会被调度一次。一旦Pod被调度分配到了某个Node上，Pod会在这个Node上运行直到自己停止或者被终止。



#### Pod lifetime

Like individual application containers, Pods are considered to be relatively ephemeral (rather than durable) entities. Pods are created, assigned a unique ID ([UID](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#uids)), and scheduled to nodes where they remain until termination (according to restart policy) or deletion.

就想独立的应用容器，Pods被认为是相对短暂的（而不是持久的）实体。Pods被创建并被分配一个UID，并被调度到Node中，它会持续在这Node中直到被终止或者删除.

If a [Node](https://kubernetes.io/docs/concepts/architecture/nodes/) dies, the Pods scheduled to that node are [scheduled for deletion](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-garbage-collection) after a timeout period.

如果Node故障挂了，被调度到该Node上的Pods会被安排计划进行删除,在一个超时周期后。

Pods do not, by themselves, self-heal. If a Pod is scheduled to a [node](https://kubernetes.io/docs/concepts/architecture/nodes/) that then fails, or if the scheduling operation itself fails, the Pod is deleted; likewise, a Pod won't survive an eviction due to a lack of resources or Node maintenance. Kubernetes uses a higher-level abstraction, called a [controller](https://kubernetes.io/docs/concepts/architecture/controller/), that handles the work of managing the relatively disposable Pod instances.

Pods的自我恢复功能不是自己实现的。如果一个Pod被调度到某个Node然后这个Pod故障了，或者调度操作失败了，这个Pod会被删除。同样的，Pod无法在资源缺失或者Node维护的时候存活。Kubernetes采用一个高层的抽象，称为Controller来处理管理这些相对而言可以用后即抛的Pods。

A given Pod (as defined by a UID) is never "rescheduled" to a different node; instead, that Pod can be replaced by a new, near-identical Pod, with even the same name if desired, but with a different UID.

一个给定的Pod(由UID来定义是否同一个Pod)只会被调度一次；但是Pod可以被一个新的，完全一样的Pod替换掉,甚至这个新的Pod有着一样的name，但是UID是不一样的。

When something is said to have the same lifetime as a Pod, such as a [volume](https://kubernetes.io/docs/concepts/storage/volumes/), that means that the thing exists as long as that specific Pod (with that exact UID) exists. If that Pod is deleted for any reason, and even if an identical replacement is created, the related thing (a volume, in this example) is also destroyed and created anew.

当我们说有些东西跟Pod的生命周期一样时，比如volume，这意味着这个东西在他关联的Pod(通过UID来指定)存在时也会一直存在。如果这个Pod因为某些原因被删除了，或者是一个完全一样的替代品Pod创建了，这个关联的东西（例子中说的是volume）也会被销毁并创建一个新的。

<img src="../../../../../assets/image-20201005215539690.png" alt="image-20201005215539690" style="zoom: 50%;" />

 **Pod diagram**

*A multi-container Pod that contains a file puller and a web server that uses a persistent volume for shared storage between the containers.*



#### Pod phase 阶段

A Pod's `status` field is a [PodStatus](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#podstatus-v1-core) object, which has a `phase` field.

The phase of a Pod is a simple, high-level summary of where the Pod is in its lifecycle. The phase is not intended to be a comprehensive rollup of observations of container or Pod state, nor is it intended to be a comprehensive state machine.

The number and meanings of Pod phase values are tightly guarded. Other than what is documented here, nothing should be assumed about Pods that have a given `phase` value.

Pod的status字段是一个PodStatus对象，它内部有一个phase字段。

Pod的phase是一个简单的，高层级的概述 of Pod当前处于生命周期的哪个阶段。phase不是为了成为Pod或者容器综合的观察指标，也不是为了成为一个综合的状态机。

Pod阶段的数量和意义是十分严谨的。除了文档列出的phase，其他phase值都不存在。

Here are the possible values for `phase`:

| Value       | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| `Pending`   | The Pod has been accepted by the Kubernetes cluster, but one or more of the containers has not been set up and made ready to run. This includes time a Pod spends waiting to be scheduled as well as the time spent downloading container images over the network.                                                                                                                             Pod被集群接受了，但是存在Container没有被组建并且没有准备好去运行。这包括Pod等待被调度的时间，以及从网络中下载镜像的时间。 |
| `Running`   | The Pod has been bound to a node, and all of the containers have been created. At least one container is still running, or is in the process of starting or restarting.                                                                                                                    Pod绑定到了Node上，并且所有容器都创建好了。至少有一个容器在运行或者处于启动或者重启中。 |
| `Succeeded` | All containers in the Pod have terminated in success, and will not be restarted.                                                              所有Pod中的容器都以成功状态结束了，并且不会重启。 |
| `Failed`    | All containers in the Pod have terminated, and at least one container has terminated in failure. That is, the container either exited with non-zero status or was terminated by the system.                                                                          Pod中所有容器都结束了，并且至少有一个容器以失败的状态结束。失败状态指容器以非零状态退出或者被系统终止。 |
| `Unknown`   | For some reason the state of the Pod could not be obtained. This phase typically occurs due to an error in communicating with the node where the Pod should be running.                                                                                             由于一些原因Pod的状态无法被获取。这个阶段通常由于跟Pod所在Node的通信问题而发生的。 |

If a node dies or is disconnected from the rest of the cluster, Kubernetes applies a policy for setting the `phase` of all Pods on the lost node to Failed.

如果一个Node挂了或者从集群中脱离连接了,Kubernetes采用一种策略来设置该丢失的Node中所有Pods的阶段为Failed。



#### Container states 容器状态

As well as the [phase](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase) of the Pod overall, Kubernetes tracks the state of each container inside a Pod. You can use [container lifecycle hooks](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/) to trigger events to run at certain points in a container's lifecycle.

Once the [scheduler](https://kubernetes.io/docs/reference/generated/kube-scheduler/) assigns a Pod to a Node, the kubelet starts creating containers for that Pod using a [container runtime](https://kubernetes.io/docs/setup/production-environment/container-runtimes). There are three possible container states: `Waiting`, `Running`, and `Terminated`.

To check the state of a Pod's containers, you can use `kubectl describe pod <name-of-pod>`. The output shows the state for each container within that Pod.

和Pod的phase一样，Kubernetes会追踪Pod中每个容器的状态。你可以使用 [container lifecycle hooks](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/) 来触发事件用于在容器的生命周期的特定时间点执行一些逻辑。

一旦调度器把Pod调度到Node上，Node上的Kubelet就会开始使用容器运行时为Pod创建容器。容器状态可能会存在三种值：`Waiting`, `Running`, and `Terminated`。

为了检查Pod中容器的状态，您可以使用`kubectl describe pod <name-of-pod>`命令。输出的内容会展示Pod中每个容器的状态。

Each state has a specific meaning:

##### `Waiting`

If a container is not in either the `Running` or `Terminated` state, it is `Waiting`. A container in the `Waiting` state is still running the operations it requires in order to complete start up: for example, pulling the container image from a container image registry, or applying [Secret](https://kubernetes.io/docs/concepts/configuration/secret/) data. When you use `kubectl` to query a Pod with a container that is `Waiting`, you also see a Reason field to summarize why the container is in that state.

如果一个容器不处于`Running or Terminated`状态，那就是处于`Waiting`状态。一个处于`Waiting`状态的容器会持续做一些操作来做启动前的准备：比如从容器镜像注册中心拉取容器镜像，或者应用Secret数据。当你使用Kubelet查询Pod里面Waiting的容器，你可以看到一个原因字段概述为什么容器处于这个状态中。

##### `Running`

The `Running` status indicates that a container is executing without issues. If there was a `postStart` hook configured, it has already executed and finished. When you use `kubectl` to query a Pod with a container that is `Running`, you also see information about when the container entered the `Running` state.

运行状态指示容器正在执行并且没有出现问题。如果你有设置了PostStart钩子，那么它是已经被执行并完成。当你使用Kubelet查询一个带有运行中状态容器的Pod时，你可以查看到一些信息关于容器何时进入的Running状态。

##### `Terminated`

A container in the `Terminated` state began execution and then either ran to completion or failed for some reason. When you use `kubectl` to query a Pod with a container that is `Terminated`, you see a reason, an exit code, and the start and finish time for that container's period of execution.

If a container has a `preStop` hook configured, that runs before the container enters the `Terminated` state.

一个处于Terminated状态的容器是先开始执行并且要不就是成功执行完成或者出现了一些异常导致失败了。当你使用Kubelet查询一个带有Terminated状态容器的Pod时候，你可以看到原因，一个退出code，以及容器执行的开始和结束时间。



#### Container restart policy

The `spec` of a Pod has a `restartPolicy` field with possible values Always, OnFailure, and Never. The default value is Always.

The `restartPolicy` applies to all containers in the Pod. `restartPolicy` only refers to restarts of the containers by the kubelet on the same node. After containers in a Pod exit, the kubelet restarts them with an exponential back-off delay (10s, 20s, 40s, …), that is capped at five minutes. Once a container has executed with no problems for 10 minutes without any problems, the kubelet resets the restart backoff timer for that container.

Pod模板的spec字段有一个restartPolicy字段，它有Always，OnFailure以及Never这三种类型。默认值是Always。

restartPolicy会应用于Pod中所有的容器。restartPolicy配置只针对Kubelet所在Node上的容器的重启行为。当Pod的中的容器退出了，Kubelet会以指数back-off延迟来重启他们（10s，20s，40s..）,最大重启延迟间隔为5分钟。一旦容器正常执行了10分钟，那么Kubelet会重置该容器对应的backoff计时器。



#### Pod conditions Pod的状况情况

A Pod has a PodStatus, which has an array of [PodConditions](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#podcondition-v1-core) through which the Pod has or has not passed:

- `PodScheduled`: the Pod has been scheduled to a node.  
- `ContainersReady`: all containers in the Pod are ready.
- `Initialized`: all [init containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) have started successfully.
- `Ready`: the Pod is able to serve requests and should be added to the load balancing pools of all matching Services.

Pod有一个PodStatus，里面有一组PodConditions，里面是一些Pod可能通过或者没有通过的测试。

- `PodScheduled`: Pod被调度到了某个Node
- `ContainersReady`: 所有在该Pod中的容器都准备好了
- `Initialized`: 所有在该Pod中的容器都成功启动了
- `Ready`: Pod现在可以支持请求服务，并且一个被添加到所有匹配的Service的LB中。

| Field name           | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| `type`               | Name of this Pod condition.  PodCondition的名称              |
| `status`             | Indicates whether that condition is applicable, with possible values "`True`", "`False`", or "`Unknown`".       标识这个Condition是否可适用的，可能的值有"`True`", "`False`", or "`Unknown`". |
| `lastProbeTime`      | Timestamp of when the Pod condition was last probed.                                                                                    PodCondition上一次探测的时间点 |
| `lastTransitionTime` | Timestamp for when the Pod last transitioned from one status to another.                                                                      PodCondition上一次发生改变的时间点 |
| `reason`             | Machine-readable, UpperCamelCase text indicating the reason for the condition's last transition.                 机器可读的，驼峰文本，标识Condition上次变化的原因 |
| `message`            | Human-readable message indicating details about the last status transition.                                                            人类可读的信息标识上次状态变换的原因 |



##### Pod readiness

**FEATURE STATE:** `Kubernetes v1.14 [stable]`

Your application can inject extra feedback or signals into PodStatus: *Pod readiness*. To use this, set `readinessGates` in the Pod's `spec` to specify a list of additional conditions that the kubelet evaluates for Pod readiness.

你的应用可以在PodStatus中注入额外的反馈或者信号：Pod就绪态。要使用Pod readiness你可以在Pod的spec字段中明确一系列额外的conditions，用于Kubelet为Pod就绪态评估。

Readiness gates are determined by the current state of `status.condition` fields for the Pod. If Kubernetes cannot find such a condition in the `status.conditions` field of a Pod, the status of the condition is defaulted to "`False`".

Readiness gates由Pod的status.condition当前状态值来决定。如果Kubernetes无法在Pod的status.condition中找到这样的一个condition，那么这个condition的值会被默认设为false。

Here is an example:

```
kind: Pod
...
spec:
  readinessGates:
    - conditionType: "www.example.com/feature-1"
status:
  conditions:
    - type: Ready                              # a built in PodCondition  内置的PodCondition
      status: "False"
      lastProbeTime: null
      lastTransitionTime: 2018-01-01T00:00:00Z
    - type: "www.example.com/feature-1"        # an extra PodCondition  额外自定义的PodCondition
      status: "False"
      lastProbeTime: null
      lastTransitionTime: 2018-01-01T00:00:00Z
  containerStatuses:
    - containerID: docker://abcd...
      ready: true
...
```



##### Status for Pod readiness 

The `kubectl patch` command does not support patching object status. To set these `status.conditions` for the pod, applications and [operators](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) should use the `PATCH` action. You can use a [Kubernetes client library](https://kubernetes.io/docs/reference/using-api/client-libraries/) to write code that sets custom Pod conditions for Pod readiness.

For a Pod that uses custom conditions, that Pod is evaluated to be ready **only** when both the following statements apply:

- All containers in the Pod are ready.
- All conditions specified in `readinessGates` are `True`.

When a Pod's containers are Ready but at least one custom condition is missing or `False`, the kubelet sets the Pod's [condition](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-condition) to `ContainersReady`.

Kubectl patch命令不支持修改Object的status。如果要给Pod的status.conditions设置，应用或者操作人需要使用PATCH操作。你可以使用Kubernetes的客户端包来编写代码设置自定义的Pod Conditions给Pod readniess。

对于使用了自定义Conditions的Pod，只有如下条件都满足才会被评估为就绪。

- 所有Pod中的容器就绪
- 所有`readinessGates`中配置的Conndition都为True

当一个Pod中的容器准备就绪了,但是自定义的Condition存在丢失或者false，Kubelet会设置Pod的Condition为`ContainersReady`。



#### Container probes

A [Probe](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#probe-v1-core) is a diagnostic performed periodically by the [kubelet](https://kubernetes.io/docs/admin/kubelet/) on a Container. To perform a diagnostic, the kubelet calls a [Handler](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#handler-v1-core) implemented by the container. There are three types of handlers:

容器探针是Kubelet用来周期性的对容器性能进行诊断。为了执行一次诊断，Kubelet调用一个由容器实现的Handler处理器。有三种处理器。

- [ExecAction](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#execaction-v1-core): Executes a specified command inside the container. The diagnostic is considered successful if the command exits with a status code of 0. 

  执行一个特定的命令在容器中，如果命令以0退出了，那么通过了诊断。

- [TCPSocketAction](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#tcpsocketaction-v1-core): Performs a TCP check against the Pod's IP address on a specified port. The diagnostic is considered successful if the port is open.

  在Pods的IP和特定端口上执行一个TCP检查。如果端口是开放的那么认为通过了诊断

- [HTTPGetAction](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#httpgetaction-v1-core): Performs an HTTP `GET` request against the Pod's IP address on a specified port and path. The diagnostic is considered successful if the response has a status code greater than or equal to 200 and less than 400.

  执行一次HTTP GET请求，在Pod的IP和特定port以及路径上。如果响应状态码在200~400之间则认为通过了诊断。

Each probe has one of three results:  每个探针都会有以下三种结果

- `Success`: The container passed the diagnostic.  容器通过了诊断
- `Failure`: The container failed the diagnostic.     容器没有通过诊断
- `Unknown`: The diagnostic failed, so no action should be taken.    诊断执行失败，使用不会采用任何行动。

The kubelet can optionally perform and react to three kinds of probes on running containers:

- `livenessProbe`: Indicates whether the container is running. If the liveness probe fails, the kubelet kills the container, and the container is subjected to its [restart policy](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy). If a Container does not provide a liveness probe, the default state is `Success`.
- `readinessProbe`: Indicates whether the container is ready to respond to requests. If the readiness probe fails, the endpoints controller removes the Pod's IP address from the endpoints of all Services that match the Pod. The default state of readiness before the initial delay is `Failure`. If a Container does not provide a readiness probe, the default state is `Success`.
- `startupProbe`: Indicates whether the application within the container is started. All other probes are disabled if a startup probe is provided, until it succeeds. If the startup probe fails, the kubelet kills the container, and the container is subjected to its [restart policy](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy). If a Container does not provide a startup probe, the default state is `Success`.

For more information about how to set up a liveness, readiness, or startup probe, see [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/).

Kubelet可以在运行的容器上有选择的执行以下三种探针并作出响应。

- `livenessProbe`：表明容器是否在运行。如果liveness 探针诊断没有通过，那么kubelet会kill掉这个容器，并且容器会受到重启策略的影响。如果容器不提供liveness探针，那么默认的状态就是success。
- `readinessProbe` ：表明容器是否准备好去响应请求。如果诊断没有通过，则Endpoints Controller会吧这个Pod的IP地址从所有匹配到的Service中移除。readiness在初始化成功之前的默认状态是Failure。如果容器没有提供readiness探针那么默认状态是Success。
- `startupProbe`: 表明容器中的应用是否启动成功了。如果这个startupProbe提供了，那么在它成功之前，所有其他的probe都会被禁用。如果startupProbe诊断没有通过，那么kubelet会kill掉这个容器，并且后续会采用restart策略。如果容器没有提供startup探针，那么默认的就是success。

关于如何设置 liveness, readiness, or startup probe可以查看[Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)。



##### When should you use a liveness probe?

**FEATURE STATE:** `Kubernetes v1.0 [stable]`

If the process in your container is able to crash on its own whenever it encounters an issue or becomes unhealthy, you do not necessarily need a liveness probe; the kubelet will automatically perform the correct action in accordance with the Pod's `restartPolicy`.

If you'd like your container to be killed and restarted if a probe fails, then specify a liveness probe, and specify a `restartPolicy` of Always or OnFailure.

如果你容器中的进程在遇到问题或者变的unhealthy情况下会自行崩溃，那么你没有必要使用liveness probe；Kubelet会自动的基于restartPolicy来作出响应的正确的行为。

如果你想要你的容器在liveness probe失败时被杀死或者重启，那么你需要指定restartPolicy为Always或者OnFailure。



##### When should you use a readiness probe?

**FEATURE STATE:** `Kubernetes v1.0 [stable]`

If you'd like to start sending traffic to a Pod only when a probe succeeds, specify a readiness probe. In this case, the readiness probe might be the same as the liveness probe, but the existence of the readiness probe in the spec means that the Pod will start without receiving any traffic and only start receiving traffic after the probe starts succeeding. If your container needs to work on loading large data, configuration files, or migrations during startup, specify a readiness probe.

If you want your container to be able to take itself down for maintenance, you can specify a readiness probe that checks an endpoint specific to readiness that is different from the liveness probe.

如果你想要在探测成功后才向Pod发送流量，那么你需要用到readiness probe。在这种情况下，readiness probe也许跟liveness probe一样。但是spec中readiness probe的存在意味着Pod在probe返回成功之前不会接受任何流量，并且只有探针诊断通过后才会接受流量。如果你的容器需要在启动的时候加载很大的数据量，配置文件或者做一些迁移，那么可以使用readiness probe。

> **Note:** If you just want to be able to drain requests when the Pod is deleted, you do not necessarily need a readiness probe; on deletion, the Pod automatically puts itself into an unready state regardless of whether the readiness probe exists. The Pod remains in the unready state while it waits for the containers in the Pod to stop.
>
> 你如果你只是想要能够在Pod被删除后排空请求，那么你不需要使用readiness probe；在删除Pod时，Pod会自动把他自己修改成unready状态，无论readiness probe是否存在。在等待Pod中容器停止的期间，Pod会一直持续处于unready状态。



##### When should you use a startup probe?

**FEATURE STATE:** `Kubernetes v1.16 [alpha]`

Startup probes are useful for Pods that have containers that take a long time to come into service. Rather than set a long liveness interval, you can configure a separate configuration for probing the container as it starts up, allowing a time longer than the liveness interval would allow.

If your container usually starts in more than `initialDelaySeconds + failureThreshold × periodSeconds`, you should specify a startup probe that checks the same endpoint as the liveness probe. The default for `periodSeconds` is 30s. You should then set its `failureThreshold` high enough to allow the container to start, without changing the default values of the liveness probe. This helps to protect against deadlocks.

Startup probe对于那些需要花费很长时间才能开始提供服务的Pod是很有用的。不需要通过设置一个很长的liveness probe探测间隔周期，你可以配置一个独立的配置项来在容器启动时进行探测，这个配置允许一个更长的比liveness probe所允许的探测间隔更加长的时间。

如果你的容器通常启动时会超出`initialDelaySeconds + failureThreshold × periodSeconds`，你应该设置一个startup probe用于检查liveness probe同一个endpoint。默认的`periodSeconds`是30s，你可以通过设置`failureThreshold`为更大的值来允许容器有更长的启动时间，而不需要修改liveness probe的默认值。这可以帮助减少死锁发生。



#### Termination of Pods

Because Pods represent processes running on nodes in the cluster, it is important to allow those processes to gracefully terminate when they are no longer needed (rather than being abruptly stopped with a `KILL` signal and having no chance to clean up).

因为Pods代表了运行在集群Node中的进程，在不需要这些进程的时候允许它们进行gracefully终止是很重要的（而不是通过突然的KILL信号来终止它们，以至于它们没有机会去做一些清理操作）。

The design aim is for you to be able to request deletion and know when processes terminate, but also be able to ensure that deletes eventually complete. When you request deletion of a Pod, the cluster records and tracks the intended grace period before the Pod is allowed to be forcefully killed. With that forceful shutdown tracking in place, the [kubelet](https://kubernetes.io/docs/reference/generated/kubelet) attempts graceful shutdown.

设计的目标是让你能够请求删除进程，并且知道什么时候这些进程终止了，以及能够确保这些删除操作最终执行成功了。当你请求删除一个Pod，集群在会记录追踪Pod优雅shutdown的周期，超过周期后Pod就允许被强行KILL掉。在forcefull shutdown追踪存在的情况下，kubelet会首先尝试优雅的shutdown。

Typically, the container runtime sends a TERM signal to the main process in each container. Many container runtimes respect the `STOPSIGNAL` value defined in the container image and send this instead of TERM. Once the grace period has expired, the KILL signal is sent to any remaining processes, and the Pod is then deleted from the [API Server](https://kubernetes.io/docs/concepts/overview/components/#kube-apiserver). If the kubelet or the container runtime's management service is restarted while waiting for processes to terminate, the cluster retries from the start including the full original grace period.

通常来说容器运行时会发送一个TERM信号到容器中的主进程。很多容器运行时会遵守容器镜像中定义的`STOPSIGNAL`值，并且发送这个来代替`TERM`。 一旦优雅shutdown的周期过了，KILL信号会被发送到所有存活着的进程，然后Pod就会从API Server中删除掉。如果kubelet或者容器运行时的管理服务重启了，在等待进程终止的时候。那么集群会重头开始重试，即从一开始的整个优雅停机流程开始。

An example flow:

1. You use the `kubectl` tool to manually delete a specific Pod, with the default grace period (30 seconds).

   你使用Kubelet工具来手动删除一个特定的Pod，以一个默认的优雅shutdown周期（30s）。

2. The Pod in the API server is updated with the time beyond which the Pod is considered "dead" along with the grace period. If you use`kubectl describe`to check on the Pod you're deleting, that Pod shows up as "Terminating". On the node where the Pod is running: as soon as the kubelet sees that a Pod has been marked as terminating (a graceful shutdown duration has been set), the kubelet begins the local Pod shutdown process.

   API Server中的Pod对象会随着grace period设定的时间推移进行更新。如果你使用`kubectl describe`来检查你所执行删除的Pod，Pod会显示`Terminating`。在Pod所运行的Node上：只要kubelet观察到Pod被标记了terminating（一个优雅shutdown周期会被设置），kubelet会开始进行本地的Pod shutdown过程。

   1. If one of the Pod's containers has defined a`preStop`hook, the kubelet runs that hook inside of the container. If the

       `preStop`hook is still running after the grace period expires, the kubelet requests a small, one-off grace period extension of 2 seconds.

      如果Pod中有容器定义了preStop钩子函数，那么Kubelet会在容器中先运行这个hook。如果preStop hook在grace perid周期到了还在运行，那么kubelet会请求进行一次小的，一次性的grace period延长，延长2s。

      > **Note:** If the `preStop` hook needs longer to complete than the default grace period allows, you must modify `terminationGracePeriodSeconds` to suit this.

   2. The kubelet triggers the container runtime to send a TERM signal to process 1 inside each container.

      kubelet触发容器运行时发送TERM信号到Pod中的容器进程。

      > **Note:** The containers in the Pod receive the TERM signal at different times and in an arbitrary order. If the order of shutdowns matters, consider using a `preStop` hook to synchronize.

3. At the same time as the kubelet is starting graceful shutdown, the control plane removes that shutting-down Pod from Endpoints (and, if enabled, EndpointSlice) objects where these represent a [Service](https://kubernetes.io/docs/concepts/services-networking/service/) with a configured [selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/). [ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) and other workload resources no longer treat the shutting-down Pod as a valid, in-service replica. Pods that shut down slowly cannot continue to serve traffic as load balancers (like the service proxy) remove the Pod from the list of endpoints as soon as the termination grace period *begins*.

   在Kubelet开始进行graceful shutdown的同时，Control Plane会吧这个Pod从Endpoints Objects中移除 where these represent a Service with a configured selector。ReplicaSets和其他工作负载资源不再把这个Pod认为是有效的，在服务中的副本。shutdown 缓慢的Pods也不能继续处理流量，因为LB（比如service proxy）在graceful shutdown period开始的时候就把Pod从LB的endpoint列表中删除了。

4. When the grace period expires, the kubelet triggers forcible shutdown. The container runtime sends `SIGKILL` to any processes still running in any container in the Pod. The kubelet also cleans up a hidden `pause` container if that container runtime uses one.

   当grace period过期了，kubelet会触发强制shutdown。容器运行时会发生SIGKILL信号到任何Pod中运行的容器中的进程。Kubelet也会清理因此的pause容器，如果这些容器运行时使用到了的话。

5. The kubelet triggers forcible removal of Pod object from the API server, by setting grace period to 0 (immediate deletion).

   Kubelet触发强制从API Server上删除Pod Object的逻辑，通过设置grace period为0（立即删除）。

6. The API server deletes the Pod's API object, which is then no longer visible from any client.

   API Server删除Pod的API  Object对象，然后任何客户端都无法看到这个对象。



##### Forced Pod termination

> **Caution:** Forced deletions can be potentially disruptive for some workloads and their Pods.

By default, all deletes are graceful within 30 seconds. The `kubectl delete` command supports the `--grace-period=<seconds>` option which allows you to override the default and specify your own value.

Setting the grace period to `0` forcibly and immediately deletes the Pod from the API server. If the pod was still running on a node, that forcible deletion triggers the kubelet to begin immediate cleanup.

> **Note:** You must specify an additional flag `--force` along with `--grace-period=0` in order to perform force deletions.

When a force deletion is performed, the API server does not wait for confirmation from the kubelet that the Pod has been terminated on the node it was running on. It removes the Pod in the API immediately so a new Pod can be created with the same name. On the node, Pods that are set to terminate immediately will still be given a small grace period before being force killed.

If you need to force-delete Pods that are part of a StatefulSet, refer to the task documentation for [deleting Pods from a StatefulSet](https://kubernetes.io/docs/tasks/run-application/force-delete-stateful-set-pod/).



##### Garbage collection of failed Pods 失效Pod的垃圾收集

For failed Pods, the API objects remain in the cluster's API until a human or [controller](https://kubernetes.io/docs/concepts/architecture/controller/) process explicitly removes them.

The control plane cleans up terminated Pods (with a phase of `Succeeded` or `Failed`), when the number of Pods exceeds the configured threshold (determined by `terminated-pod-gc-threshold` in the kube-controller-manager). This avoids a resource leak as Pods are created and terminated over time.

对于失效了的Pods，API Objects会任然呆在集群的API服务器中，直到一个人活着Controller进程显式的删除他们。

Control Plane清理终止了的Pods（处于Succeeded或Failed阶段），当Pods的数量超过配置的阈值（由kube-controller-manager中的`terminated-pod-gc-threshold`参数配置）。垃圾回收避免了随着时间推进Pod不断创建和终止导致的资源泄漏问题。