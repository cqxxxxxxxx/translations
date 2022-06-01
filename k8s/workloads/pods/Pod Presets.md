[TOC]



# Pod Presets

**FEATURE STATE:** `Kubernetes v1.6 [alpha]`

This page provides an overview of PodPresets, which are objects for injecting certain information into pods at creation time. The information can include secrets, volumes, volume mounts, and environment variables.

这一页提供一个PodPreset的概述，PodPreset是一个Object用来在Pod创建时注入特定的信息到Pod中。信息可以是secrets，卷，卷挂载和环境变量。



## Understanding Pod presets

A PodPreset is an API resource for injecting additional runtime requirements into a Pod at creation time. You use [label selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors) to specify the Pods to which a given PodPreset applies.

Using a PodPreset allows pod template authors to not have to explicitly provide all information for every pod. This way, authors of pod templates consuming a specific service do not need to know all the details about that service.

PodPreset是一个API资源用于注入额外的运行时需求的信息到Pod中，在Pod创建时。您使用label selectors来指定PodPreset应用于哪些个Pods。

使用PodPreset可以让Pod模板作者避免为每个Pod显式的提供信息。这样使用特定服务的Pod模板作者不需要知道这个服务的所有内部细节。



## Enable PodPreset in your cluster  集群开启PodPreset支持

In order to use Pod presets in your cluster you must ensure the following:

1. You have enabled the API type `settings.k8s.io/v1alpha1/podpreset`. For example, this can be done by including `settings.k8s.io/v1alpha1=true` in the `--runtime-config` option for the API server. In minikube add this flag `--extra-config=apiserver.runtime-config=settings.k8s.io/v1alpha1=true` while starting the cluster.

2. You have enabled the admission controller named `PodPreset`. One way to doing this is to include `PodPreset` in the `--enable-admission-plugins` option value specified for the API server. For example, if you use Minikube, add this flag:

   ```shell
   --extra-config=apiserver.enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota,PodPreset
   ```

   while starting your cluster.

      

## How it works  PodPreset如何工作

Kubernetes provides an admission controller (`PodPreset`) which, when enabled, applies Pod Presets to incoming pod creation requests. When a pod creation request occurs, the system does the following:

1. Retrieve all `PodPresets` available for use.
2. Check if the label selectors of any `PodPreset` matches the labels on the pod being created.
3. Attempt to merge the various resources defined by the `PodPreset` into the Pod being created.
4. On error, throw an event documenting the merge error on the pod, and create the pod *without* any injected resources from the `PodPreset`.
5. Annotate the resulting modified Pod spec to indicate that it has been modified by a `PodPreset`. The annotation is of the form `podpreset.admission.kubernetes.io/podpreset-<pod-preset name>: "<resource version>"`.

Each Pod can be matched by zero or more PodPresets; and each PodPreset can be applied to zero or more Pods. When a PodPreset is applied to one or more Pods, Kubernetes modifies the Pod Spec. For changes to `env`, `envFrom`, and `volumeMounts`, Kubernetes modifies the container spec for all containers in the Pod; for changes to `volumes`, Kubernetes modifies the Pod Spec.

> **Note:**
>
> A Pod Preset is capable of modifying the following fields in a Pod spec when appropriate:
>
> - The `.spec.containers` field
> - The `.spec.initContainers` field



### Disable Pod Preset for a specific pod  对特定Pod禁用PodPreset

There may be instances where you wish for a Pod to not be altered by any Pod preset mutations. In these cases, you can add an annotation in the Pod's `.spec` of the form: `podpreset.admission.kubernetes.io/exclude: "true"`.