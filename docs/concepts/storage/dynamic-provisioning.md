---
approvers:
- saad-ali
cn-approvers:
- lichuqiang
title: 动态卷供应（Provision）
---
<!--
---
approvers:
- saad-ali
title: Dynamic Volume Provisioning
---
-->

{% capture overview %}

<!--
Dynamic volume provisioning allows storage volumes to be created on-demand.
Without dynamic provisioning, cluster administrators have to manually make
calls to their cloud or storage provider to create new storage volumes, and
then create [`PersistentVolume` objects](/docs/concepts/storage/persistent-volumes/)
to represent them in Kubernetes. The dynamic provisioning feature eliminates
the need for cluster administrators to pre-provision storage. Instead, it
automatically provisions storage when it is requested by users.
-->
动态卷供应允许按需创建存储卷。 如果没有动态供应，集群管理员必须手动调用其云服务或存储提供商（的接口）来创建新的存储卷，
然后在 Kubernetes 中创建 [`PersistentVolume` 对象](/docs/concepts/storage/persistent-volumes/)
来代表这些卷。 动态供应特性使得集群管理员不必预先准备存储卷。 相反，它在用户请求时自动提供存储。

{% endcapture %}

{:toc}

{% capture body %}

<!--
## Background

The implementation of dynamic volume provisioning is based on the API object `StorageClass`
from the API group `storage.k8s.io`. A cluster administrator can define as many
`StorageClass` objects as needed, each specifying a *volume plugin* (aka
*provisioner*) that provisions a volume and the set of parameters to pass to
that provisioner when provisioning.
-->
## 背景

动态卷供应的实现基于 `storage.k8s.io` API 组 中的 `StorageClass` API 对象。
集群管理员可以根据需要，定义任意数量的 `StorageClass` 对象，
每个 `StorageClass` 对象中指定一个 *卷插件*（又名*提供商*），
以及动态供应时传递到提供商的一系列参数。
<!--
A cluster administrator can define and expose multiple flavors of storage (from
the same or different storage systems) within a cluster, each with a custom set
of parameters. This design also ensures that end users don’t have to worry
about the the complexity and nuances of how storage is provisioned, but still
have the ability to select from multiple storage options.
-->

集群管理员可以在集群中定义和暴露多种不同的存储（来自相同或不同的存储系统），
每种存储都有自定义的参数集。这样的设计也保证了最终用户不必关心存储供应的复杂性和细微差别，
但仍有能力从多个存储选项中进行选择。

<!--
More information on storage classes can be found
[here](/docs/concepts/storage/persistent-volumes/#storageclasses).
-->
更多关于存储类（storage class）的信息请参考
[这里](/docs/concepts/storage/persistent-volumes/#storageclasses)。

<!--
## Enabling Dynamic Provisioning

To enable dynamic provisioning, a cluster administrator needs to pre-create
one or more StorageClass objects for users.
-->
## 启用动态供应

为启用动态供应，集群管理员需要预先为用户创建一个或多个 StorageClass 对象。
<!--
StorageClass objects define which provisioner should be used and what parameters
should be passed to that provisioner when dynamic provisioning is invoked.
The following manifest creates a storage class "slow" which provisions standard
disk-like persistent disks.
-->
StorageClass 对象定义了调用动态供应时所使用的提供商和所传入的参数。
下面的声明创建了一个名为 “slow” 的存储类，它提供类似标准磁盘的持久化磁盘。

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
```

<!--
The following manifest creates a storage class "fast" which provisions
SSD-like persistent disks.
-->

下面的声明创建了一个名为 “fast” 的存储类，它提供类似 SSD 的持久化磁盘。

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```

<!--
## Using Dynamic Provisioning

Users request dynamically provisioned storage by including a storage class in
their `PersistentVolumeClaim`. Before Kubernetes v1.6, this was done via the
`volume.beta.kubernetes.io/storage-class` annotation. However, this annotation
is deprecated since v1.6. Users now can and should instead use the
`storageClassName` field of the `PersistentVolumeClaim` object. The value of
this field must match the name of a `StorageClass` configured by the
administrator (see [below](#enabling-dynamic-provisioning)).
-->
## 使用动态供应

用户通过在 `PersistentVolumeClaim` 中包含存储类来请求使用动态供应的存储。
在 Kubernetes 1.6 版本之前，存储类信息通过
`volume.beta.kubernetes.io/storage-class` 注解来指定。 然而，从 1.6 版本开始，
该注解不再推荐使用。 用户现在可以并且应该使用 `PersistentVolumeClaim` 对象的
`storageClassName` 字段来代替该注解。 该字段的值须与管理员配置的
`StorageClass` 名称相匹配 (参考 [这里](#启用动态供应))。

<!--
To select the “fast” storage class, for example, a user would create the
following `PersistentVolumeClaim`:
-->
例如，为选择 “fast” 存储类，用户应该创建以下 `PersistentVolumeClaim`：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim1
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast
  resources:
    requests:
      storage: 30Gi
```

<!--
This claim results in an SSD-like Persistent Disk being automatically
provisioned. When the claim is deleted, the volume is destroyed.
-->
该 claim 使得一个 类似 SSD 的持久磁盘被自动提供。 当 claim 被删除时，卷也被相应销毁。

<!--
## Defaulting Behavior

Dynamic provisioning can be enabled on a cluster such that all claims are
dynamically provisioned if no storage class is specified. A cluster administrator
can enable this behavior by:
-->
## 默认行为

动态供应可以按照这样的方式启用： claim 未指定存储类的情况下，会动态配置（为默认存储类）。
群集管理员可以通过以下方式启用此行为：

<!--
- Marking one `StorageClass` object as *default*;
- Making sure that the [`DefaultStorageClass` admission controller](/docs/admin/admission-controllers/#defaultstorageclass)
  is enabled on the API server.
-->
- 将一个 `StorageClass` 对象标记为 *default*，
- 保证 [`DefaultStorageClass` 准入控制器](/docs/admin/admission-controllers/#defaultstorageclass)
  在 API 服务器上启用。

<!--
An administrator can mark a specific `StorageClass` as default by adding the
`storageclass.kubernetes.io/is-default-class` annotation to it.
When a default `StorageClass` exists in a cluster and a user creates a
`PersistentVolumeClaim` with `storageClassName` unspecified, the
`DefaultStorageClass` admission controller automatically adds the
`storageClassName` field pointing to the default storage class.
-->
管理员可以通过为特定的 `StorageClass` 添加 `storageclass.kubernetes.io/is-default-class`
注解，将其标记为默认存储类。 当集群中存在一个默认 `StorageClass`，且用户创建了一个未指定
`storageClassName` 的 `PersistentVolumeClaim` 时，`DefaultStorageClass` 准入控制器自动为其添加 `storageClassName` 字段，字段指向默认存储类。

<!--
Note that there can be at most one *default* storage class on a cluster, or
a `PersistentVolumeClaim` without `storageClassName` explicitly specified cannot
be created.
-->
注意集群中最多只能有一个 *default* 存储类，否则将无法创建未明确指定 `storageClassName`
的 `PersistentVolumeClaim`。

{% endcapture %}

{% include templates/concept.md %}
