```

```

[TOC]

### Init Containers

This page provides an overview of init containers: specialized containers that run before app containers in a [Pod](https://kubernetes.io/docs/concepts/workloads/pods/). Init containers can contain utilities or setup scripts not present in an app image.

You can specify init containers in the Pod specification alongside the `containers` array (which describes app containers).

这一页提供了一个init containers的一个概念：一种在Pod中的特殊的containers, 它运行在app container之前。init containers可以包含不存在app镜像中的工具或者安装脚本。

你可以在Pod的Specification中与containers数组同一个级别上指定你的init容器。



#### Understanding init containers

A [Pod](https://kubernetes.io/docs/concepts/workloads/pods/) can have multiple containers running apps within it, but it can also have one or more init containers, which are run before the app containers are started.

Pod内部可以有多个应用容器，同时也可以有一或多个init容器，init容器会运行在app容器运行之前。

Init containers are exactly like regular containers, except:

- Init containers always run to completion.
- Each init container must complete successfully before the next one starts.

Init容器跟普通的容器几乎一模一样，除了：

- init容器总是运行到完成
- 每个init容器都必须成功完成之后，下一个init容器才能启动。

If a Pod's init container fails, the kubelet repeatedly restarts that init container until it succeeds. However, if the Pod has a `restartPolicy` of Never, and an init container fails during startup of that Pod, Kubernetes treats the overall Pod as failed.

To specify an init container for a Pod, add the `initContainers` field into the Pod specification, as an array of objects of type [Container](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#container-v1-core), alongside the app `containers` array. The status of the init containers is returned in `.status.initContainerStatuses` field as an array of the container statuses (similar to the `.status.containerStatuses` field).

如果Pod的init容器失败了，kubelet会持续的重启它直到成功。但是如果Pod的restartPolicy是Never，并且init容器在启动时失败了，那么Kubernetes把这个Pod当做failed。

要为Pod设置init容器，添加initContainers字段到Pod的specification中，以一组Container类型的Object，与应用容器配置项containers平级。init容器的状态在`.status.initContainerStatuses` 字段中，以一个数组返回容器的状态（跟`.status.containerStatuses`类似 ）。

##### Differences from regular containers

Init containers support all the fields and features of app containers, including resource limits, volumes, and security settings. However, the resource requests and limits for an init container are handled differently, as documented in [Resources](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#resources).

Also, init containers do not support `lifecycle`, `livenessProbe`, `readinessProbe`, or `startupProbe` because they must run to completion before the Pod can be ready.

If you specify multiple init containers for a Pod, Kubelet runs each init container sequentially. Each init container must succeed before the next can run. When all of the init containers have run to completion, Kubelet initializes the application containers for the Pod and runs them as usual.



#### Using init containers

Because init containers have separate images from app containers, they have some advantages for start-up related code:

- Init containers can contain utilities or custom code for setup that are not present in an app image. For example, there is no need to make an image `FROM` another image just to use a tool like `sed`, `awk`, `python`, or `dig` during setup.
- The application image builder and deployer roles can work independently without the need to jointly build a single app image.
- Init containers can run with a different view of the filesystem than app containers in the same Pod. Consequently, they can be given access to [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) that app containers cannot access.
- Because init containers run to completion before any app containers start, init containers offer a mechanism to block or delay app container startup until a set of preconditions are met. Once preconditions are met, all of the app containers in a Pod can start in parallel.
- Init containers can securely run utilities or custom code that would otherwise make an app container image less secure. By keeping unnecessary tools separate you can limit the attack surface of your app container image.

因为init容器的镜像跟app容器不同，他们有一些跟启动相关的代码功能特点。

- init容器可以包含那些APP镜像中不存在的工具或者自定义代码来帮助安装设置。比如，如果只是为了使用`sed`, `awk`, `python`, or `dig`这些命令在安装阶段，那么没有必要去把APP镜像集成From另外一个带有这些命令的镜像，你只要使用一个带有这些命令的init容器来做安装设置就好了。

- app镜像的构建和部署可以分开独自工作，没有必要把他们融合到一个单一的APP镜像中，即init容器用于构建，app容器用于部署。

- Init容器可以以不同于APP容器的文件系统视图启动。因此，他们可以被给与权限去访问Secrets，而app容器不能访问。

- 因为APP容器在初始化容器运行成功之后才启动，init容器提供一个机制来阻止或者延迟app容器的启动，直到一系列前置条件符合了。一旦前置条件满足了，init容器会执行完毕，然后app容器可以并行的启动了。

- init容器可以安全的运行工具和自定义代码，这些代码可能会导致app容器变的不安全。通过把这些不必须的工具从app容器镜像中分离开来，你可以有效的避免app容器被攻击。

  

##### Examples

https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#examples





#### Detailed behavior  具体行为[ ](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#detailed-behavior)

During Pod startup, the kubelet delays running init containers until the networking and storage are ready. Then the kubelet runs the Pod's init containers in the order they appear in the Pod's spec.

在Pod启动的期间，Kubelet会在网络和存储准备好后才启动init容器。然后kubelet按照Pod Spec中定义的顺序来运行init容器。

Each init container must exit successfully before the next container starts. If a container fails to start due to the runtime or exits with failure, it is retried according to the Pod `restartPolicy`. However, if the Pod `restartPolicy` is set to Always, the init containers use `restartPolicy` OnFailure.

每一个init容器必须成功的退出，然后下一个init容器才能继续运行。如果容器由于运行时的原因或者以错误退出，kubelet会基于Pod的restartPolicy来重试。但是，如果Pod的restartPolicy设置成Always，init容器会在失败时使用该策略。

A Pod cannot be `Ready` until all init containers have succeeded. The ports on an init container are not aggregated under a Service. A Pod that is initializing is in the `Pending` state but should have a condition `Initialized` set to true.

一个Pod不能进入Ready状态直到所有init容器都执行成功了。init容器的端口不会在Service中聚合。正在初始化中的 Pod 处于 `Pending` 状态， 但会将状况（Condition）的 `Initializing` 设置为 true。

If the Pod [restarts](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#pod-restart-reasons), or is restarted, all init containers must execute again.

如果Pod重启，所有init容器都必须再次执行。

Changes to the init container spec are limited to the container image field. Altering an init container image field is equivalent to restarting the Pod.

init容器Spec的变动只允许是在容器image字段上。修改这个字段就相当于重启Pod。

Because init containers can be restarted, retried, or re-executed, init container code should be idempotent. In particular, code that writes to files on `EmptyDirs` should be prepared for the possibility that an output file already exists.

因为init容器可以被重启，重试或者重新执行，init容器的代码必须是幂等的。尤其是，那些基于 `EmptyDirs` 写文件的代码必须对文件已存在的情况做处理准备。

Init containers have all of the fields of an app container. However, Kubernetes prohibits `readinessProbe` from being used because init containers cannot define readiness distinct from completion. This is enforced during validation.

init容器有app容器的所有字段，但是Kubernetes禁止了`readinessProbe`字段的使用，因为init容器不能定义readiness从comletion区分开来。这个是强制校验的。

Use `activeDeadlineSeconds` on the Pod and `livenessProbe` on the container to prevent init containers from failing forever. The active deadline includes init containers.

使用Pod的`activeDeadlineSeconds`和容器的 `livenessProbe`探针来避免init容器无线失败重启。active deadline包括init容器。

The name of each app and init container in a Pod must be unique; a validation error is thrown for any container sharing a name with another.

Pod中init和app容器的name必须是唯一的；如果存在一样的name，那么一个校验错误会被抛出来。



##### Resources 资源

Given the ordering and execution for init containers, the following rules for resource usage apply:

- The highest of any particular resource request or limit defined on all init containers is the *effective init request/limit*
- The Pod's effective request/limit for a resource is the higher of:
  - the sum of all app containers request/limit for a resource
  - the effective init request/limit for a resource
- Scheduling is done based on effective requests/limits, which means init containers can reserve resources for initialization that are not used during the life of the Pod.
- The QoS (quality of service) tier of the Pod's *effective QoS tier* is the QoS tier for init containers and app containers alike.

Quota and limits are applied based on the effective Pod request and limit.

Pod level control groups (cgroups) are based on the effective Pod request and limit, the same as the scheduler.



##### Pod restart reasons Pod重启的原因

A Pod can restart, causing re-execution of init containers, for the following reasons:

- A user updates the Pod specification, causing the init container image to change. Any changes to the init container image restarts the Pod. App container image changes only restart the app container.

  用户更新了Pod的spec造成了init容器镜像发生了改动。任何导致init容器镜像发生变动的都会重启Pod。App容器镜像的修改只会重启APP容器。

- The Pod infrastructure container is restarted. This is uncommon and would have to be done by someone with root access to nodes.

  Pod的基础设施容器重启。这是不常见的，必须由某些有Node的root访问权限的人来执行。

- All containers in a Pod are terminated while `restartPolicy` is set to Always, forcing a restart, and the init container completion record has been lost due to garbage collection.

  所有在Pod中的容器都终止了，并且restartPolicy设置是Always，强制进行一个重启，并且init容器的完成记录由于GC丢失了。

