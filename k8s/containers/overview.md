# Containers

Each container that you run is repeatable; the standardization from having dependencies included means that you get the same behavior wherever you run it.

Containers decouple applications from underlying host infrastructure. This makes deployment easier in different cloud or OS environments.

每一个容器你运行的都是可以重复的；这种把依赖包含在内的标准化意味着你无论你在哪里运行容器，你都能得到一致的行为。



## Container images

A [container image](https://kubernetes.io/docs/concepts/containers/images/) is a ready-to-run software package, containing everything needed to run an application: the code and any runtime it requires, application and system libraries, and default values for any essential settings.

By design, a container is immutable: you cannot change the code of a container that is already running. If you have a containerized application and want to make changes, you need to build a new image that includes the change, then recreate the container to start from the updated image.

容器镜像是一个准备好可以运行的软件包，包含着应用运行的所有必须的东西：代码以及运行时需要的对象，应用和系统库，还有配置所需要的默认值。

根据设计，容器是不可变的：你不能修改运行中容器中的code。如果你有一个容器化的应用并想要做一些修改，你需要构建一个包含这些修改的新的镜像，然后基于新的镜像重新创建运行这个容器。



## Container runtimes

The container runtime is the software that is responsible for running containers.

Kubernetes supports several container runtimes: [Docker](https://docs.docker.com/engine/), [containerd](https://containerd.io/docs/), [CRI-O](https://cri-o.io/#what-is-cri-o), and any implementation of the [Kubernetes CRI (Container Runtime Interface)](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md).

容器运行时是一个软件负责运行容器。

Kubernetes支持多种容器运行时：[Docker](https://docs.docker.com/engine/), [containerd](https://containerd.io/docs/), [CRI-O](https://cri-o.io/#what-is-cri-o), 以及其他任何只要实现了 [Kubernetes CRI (Container Runtime Interface)](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md).

