---
title: 构建高可用集群
---

<!--
## Introduction
-->
## 简介

<!--
This document describes how to build a high-availability (HA) Kubernetes cluster.  This is a fairly advanced topic.
-->
本文描述了如何构建一个高可用（high-availability, HA）的Kubernetes集群。这是一个非常高级的主题。
<!--
Users who merely want to experiment with Kubernetes are encouraged to use configurations that are simpler to set up such
as [Minikube](/docs/getting-started-guides/minikube/)
or try [Google Container Engine](https://cloud.google.com/container-engine/) for hosted Kubernetes.
-->
对于仅希望使用 Kubernetes 进行试验的用户，推荐使用更简单的配置工具进行搭建，例如：
[Minikube](/docs/getting-started-guides/minikube/)，或者尝试使用 [Google Container Engine](https://cloud.google.com/container-engine/) 来运行 Kubernetes。
<!--
Also, at this time high availability support for Kubernetes is not continuously tested in our end-to-end (e2e) testing.  We will
be working to add this continuous testing, but for now the single-node master installations are more heavily tested.
-->
此外，当前在我们的端到端（e2e）测试环境中，没有对 Kubernetes 高可用的支持进行连续测试。我们将会增加这个连续测试项，但当前对单节点 master 的安装测试得更加严格。

* TOC
{:toc}
<!--
## Overview
-->
## 概览

<!--
Setting up a truly reliable, highly available distributed system requires a number of steps. It is akin to
wearing underwear, pants, a belt, suspenders, another pair of underwear, and another pair of pants.  We go into each
of these steps in detail, but a summary is given here to help guide and orient the user.
-->
搭建一个正真可靠，高度可用的分布式系统需要若干步骤。这类似于穿上内衣、裤子、皮带

背带，另一套内衣和另一套裤子。我们会详细介绍每一个步骤，但先在这里给出一个总结来帮助指导用户。

<!--
The steps involved are as follows:

   * [Creating the reliable constituent nodes that collectively form our HA master implementation.](#reliable-nodes)
   * [Setting up a redundant, reliable storage layer with clustered etcd.](#establishing-a-redundant-reliable-data-storage-layer)
   * [Starting replicated, load balanced Kubernetes API servers](#replicated-api-servers)
   * [Setting up master-elected Kubernetes scheduler and controller-manager daemons](#master-elected-components)
       -->
       相关步骤如下：

   * [创建可靠的组成节点，共同形成我们的高可用主节点实现](#可靠的节点)
   * [使用 etcd 集群，搭建一个冗余的，可靠的存储层](#建立一个冗余的，可靠的存储层)
   * [启动具有备份和负载均衡能力的 Kubernetes API 服务](#复制的API服务)
   * [部署具有 master 选举功能的 Kubernetes scheduler 和 controller-manager 守护程序](#进行master选举的组件)
       <!--
         Here's what the system should look like when it's finished:
         -->
       系统完成时看起来应该像这样：

![High availability Kubernetes diagram](/images/docs/ha.svg)

<!--
## Initial set-up
-->
## 初始配置

<!--
The remainder of this guide assumes that you are setting up a 3-node clustered master, where each machine is running some flavor of Linux.
-->
本文假设您正在搭建一个包含 3 个节点的 master 节点集群，每个节点上都运行着某个发行版的 Linux 系统。
<!--
Examples in the guide are given for Debian distributions, but they should be easily adaptable to other distributions.
-->
虽然本指南中的示例使用的是 Debian 发行版，但是应该也可以轻松移植到其他发行版上。
<!--
Likewise, this set up should work whether you are running in a public or private cloud provider, or if you are running
on bare metal.
-->
同样的，不管在公有云还是私有云亦或是裸机上，该配置应该都可以运行。

<!--
The easiest way to implement an HA Kubernetes cluster is to start with an existing single-master cluster.  The
instructions at [https://get.k8s.io](https://get.k8s.io)
describe easy installation for single-master clusters on a variety of platforms.
-->
从一个现成的单 master 节点集群开始是实现一个高可用 Kubernetes 集群的最简单的方法。这篇指导 [https://get.k8s.io](https://get.k8s.io) 描述了在多种平台上方便的安装一个单 master 节点集群的方法。
<!--
## Reliable nodes
-->
## 可靠的节点

<!--
On each master node, we are going to run a number of processes that implement the Kubernetes API.  The first step in making these reliable is
to make sure that each automatically restarts when it fails.  To achieve this, we need to install a process watcher.  We choose to use
the `kubelet` that we run on each of the worker nodes.  This is convenient, since we can use containers to distribute our binaries, we can
establish resource limits, and introspect the resource usage of each daemon.  Of course, we also need something to monitor the kubelet
itself (insert who watches the watcher jokes here).  For Debian systems, we choose monit, but there are a number of alternate
choices. For example, on systemd-based systems (e.g. RHEL, CentOS), you can run 'systemctl enable kubelet'.
-->
我们在每个 master 节点上都将运行数个实现 Kubernetes API 的进程。保证它们可靠的第一步是使其在发生故障时，每一个进程都可以自动重启。为了实现这个目标，我们需要安装一个进程监视器。我们选择使用在每个 worker 节点上都会运行的 `kubelet` 进程。这会带来便利性，因为我们使用了容器来分发我们的二进制文件，所以我们能够为每一个守护程序设置资源限制并监视它们的消耗的资源。当然，我们也需要一些手段来监控 kubelet 本身（在此监测监控者本身是一个有趣的话题）。对于 Debian 系统我们选择使用 monit 来监控 kubelet，但也有许多其他的工具可供选择。例如在基于 systemd 的系统上（如 RHEL、CentOS），您可以运行 'systemctl enable kubelet'。

<!--
If you are extending from a standard Kubernetes installation, the `kubelet` binary should already be present on your system.  You can run
`which kubelet` to determine if the binary is in fact installed.  If it is not installed,
you should install the [kubelet binary](https://storage.googleapis.com/kubernetes-release/release/v0.19.3/bin/linux/amd64/kubelet), the
[kubelet init file](http://releases.k8s.io/{{page.githubbranch}}/cluster/saltbase/salt/kubelet/initd) and [default-kubelet](/docs/admin/high-availability/default-kubelet)
scripts.
-->
如果您是从标准的 Kubernetes 安装扩展而来，那么 `kubelet` 二进制文件应该已经存在于您的系统中。您可以运行 `which kubelet` 来判断是否确实安装了这个二进制文件。如果没有安装的话，您应该手动安装 [kubelet binary](https://storage.googleapis.com/kubernetes-release/release/v0.19.3/bin/linux/amd64/kubelet), 
[kubelet init file](http://releases.k8s.io/{{page.githubbranch}}/cluster/saltbase/salt/kubelet/initd) 和 [default-kubelet](/docs/admin/high-availability/default-kubelet) 脚本。
<!--
If you are using monit, you should also install the monit daemon (`apt-get install monit`) and the [monit-kubelet](/docs/admin/high-availability/monit-kubelet) and
[monit-docker](/docs/admin/high-availability/monit-docker) configs.
-->
如果使用 monit，您还需要安装 monit 守护程序（`apt-get install monit`）以及 [monit-kubelet](/docs/admin/high-availability/monit-kubelet) 和
[monit-docker](/docs/admin/high-availability/monit-docker) 配置。
<!--
On systemd systems you `systemctl enable kubelet` and `systemctl enable docker`.
-->
在使用 systemd 的系统上，您可以执行 `systemctl enable kubelet` 和 `systemctl enable docker`。

<!--
## Establishing a redundant, reliable data storage layer
-->
## 建立一个冗余的，可靠的存储层

<!--
The central foundation of a highly available solution is a redundant, reliable storage layer.  The number one rule of high-availability is
to protect the data.  Whatever else happens, whatever catches on fire, if you have the data, you can rebuild.  If you lose the data, you're
done.
-->
高可用方案的重心是要有一个冗余可靠的存储层。高可用的一等规则是保护数据。不管发生了什么，不管什么着了火，只要还有数据，您就可以重建。如果丢掉了数据，您就完了。

<!--
Clustered etcd already replicates your storage to all master instances in your cluster.  This means that to lose data, all three nodes would need
to have their physical (or virtual) disks fail at the same time.  The probability that this occurs is relatively low, so for many people
running a replicated etcd cluster is likely reliable enough.  You can add additional reliability by increasing the
size of the cluster from three to five nodes.  If that is still insufficient, you can add
[even more redundancy to your storage layer](#even-more-reliable-storage).
-->
集群化的 etcd 已经把您存储的数据复制到了您集群中的所有 master 节点实例上。这意味着如果要想丢失数据，三个节点的物理（或虚拟）硬盘需要全部同时故障。这种情况发生的概率是比较低的，所以对于许多人来说，运行一个复制的 etcd 集群可能已经足够的可靠了。您可以将 etcd 集群中节点的数量从3个增大到5个来增加集群的可靠性。如果那样还不够的话，您可以向[存储层增加更多的冗余](#更加可靠的存储)。

<!--
### Clustering etcd
-->
### 集群化etcd

<!--
The full details of clustering etcd are beyond the scope of this document, lots of details are given on the
[etcd clustering page](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/clustering.md).  This example walks through
a simple cluster set up, using etcd's built in discovery to build our cluster.
-->
集群化 etcd 的完整细节超出了本文范围，您可以在 [etcd clustering page](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/clustering.md) 找到许多详细内容。这个例子仅走查一个简单的集群建立过程，使用etcd内置的发现功能来构建我们的集群。

<!--
First, hit the etcd discovery service to create a new token:

```shell
curl https://discovery.etcd.io/new?size=3
```
-->
首先，调用 etcd 发现服务来创建一个新令牌:

```shell
curl https://discovery.etcd.io/new?size=3
```

<!--
On each node, copy the [etcd.yaml](/docs/admin/high-availability/etcd.yaml) file into `/etc/kubernetes/manifests/etcd.yaml`
-->
在每个节点上，拷贝 [etcd.yaml](/docs/admin/high-availability/etcd.yaml) 文件到`/etc/kubernetes/manifests/etcd.yaml`。

<!--
The kubelet on each node actively monitors the contents of that directory, and it will create an instance of the `etcd`
server from the definition of the pod specified in `etcd.yaml`.
-->
每个节点上的kubelet会动态的监控这个目录的内容，并且会按照 `etcd.yaml` 里对pod的定义创建一个 `etcd` 服务的实例。

<!--
Note that in `etcd.yaml` you should substitute the token URL you got above for `${DISCOVERY_TOKEN}` on all three machines,
and you should substitute a different name (e.g. `node-1`) for `${NODE_NAME}` and the correct IP address
for `${NODE_IP}` on each machine.
-->
请注意，您应该使用上文中获取的令牌 URL 替换全部三个节点上 `etcd.yaml` 中的`${DISCOVERY_TOKEN}` 项。同时还应该将每个节点上的 `${NODE_NAME}` 替换为一个不同的名字（例如：`node-1`），并将 `${NODE_IP}` 替换为正确的IP地址。

<!--
#### Validating your cluster
-->
#### 验证您的集群

<!--
Once you copy this into all three nodes, you should have a clustered etcd set up.  You can validate on master with
```shell
kubectl exec < pod_name > etcdctl member list
```

and

```shell
kubectl exec < pod_name > etcdctl cluster-health
```
-->
如果已经将这个文件拷贝到所有三个节点，您应该已经搭建起了一个集群化的etcd。您可以在主节点上进行验证：
```shell
kubectl exec < pod_name > etcdctl member list
```

和

```shell
kubectl exec < pod_name > etcdctl cluster-health
```

<!--
You can also validate that this is working with `etcdctl set foo bar` on one node, and `etcdctl get foo`
on a different node.
-->
您也可以在一个节点上运行 `etcdctl set foo bar`，在另一个节点上运行`etcdctl get foo`来验证集群是否工作正常。

<!--
### Even more reliable storage
-->
### 更加可靠的存储

<!--
Of course, if you are interested in increased data reliability, there are further options which makes the place where etcd
installs it's data even more reliable than regular disks (belts *and* suspenders, ftw!).
-->
当然，如果您对增加数据的可靠性感兴趣，这里还有一些更深入的选项可以使 etcd 把它的数据存放在比常规硬盘更可靠的地方（裤带和背带，ftw!）。

<!--
If you use a cloud provider, then they usually provide this
for you, for example [Persistent Disk](https://cloud.google.com/compute/docs/disks/persistent-disks) on the Google Cloud Platform.  These
are block-device persistent storage that can be mounted onto your virtual machine. Other cloud providers provide similar solutions.
-->
如果您使用云服务，那么您的提供商通常会为您提供这个特性，例如 Google Cloud Platform 上的 [Persistent Disk](https://cloud.google.com/compute/docs/disks/persistent-disks) 。它们是可以挂载到您的虚拟机中的块设备持久化存储。其他的云服务提供商提供了类似的解决方案。

<!--
If you are running on physical machines, you can also use network attached redundant storage using an iSCSI or NFS interface.
Alternatively, you can run a clustered file system like Gluster or Ceph.  Finally, you can also run a RAID array on each physical machine.
-->
如果运行于物理机之上，您仍然可以使用 iSCSI 或者 NFS 接口通过网络来连接冗余存储。
此外，您还可以运行一个集群文件系统，比如 Gluster 或者 Ceph。最后，您还可以在您的每个物理机器上运行 RAID 矩阵。

<!--
Regardless of how you choose to implement it, if you chose to use one of these options, you should make sure that your storage is mounted
to each machine.  If your storage is shared between the three masters in your cluster, you should create a different directory on the storage
for each node.  Throughout these instructions, we assume that this storage is mounted to your machine in `/var/etcd/data`
-->
不管您选择如何实现，如果已经选择了使用其中的一个选项，那么您应该保证您的存储被挂载到了每一台机器上。如果您的存储在集群中的三个 master 节点之间共享，那么您应该在存储上为每一个节点创建一个不同的目录。对于所有的这些指导，我们都假设这个存储被挂载到您机器上的 `/var/etcd/data` 路径。

<!--
## Replicated API Servers
-->
## 多个实例的API Server

<!--
Once you have replicated etcd set up correctly, we will also install the apiserver using the kubelet.
-->
在正确搭建多个实例的 etcd 之后，我们还需要使用 kubelet 安装 apiserver。

<!--
### Installing configuration files
-->

### 安装配置文件

<!--
First you need to create the initial log file, so that Docker mounts a file instead of a directory:

```shell
touch /var/log/kube-apiserver.log
```
-->
首先，您需要创建初始的日志文件，这样 Docker 才会挂载一个文件而不是一个目录：
```shell
touch /var/log/kube-apiserver.log
```
<!--
Next, you need to create a `/srv/kubernetes/` directory on each node.  This directory includes:

   * basic_auth.csv  - basic auth user and password
   * ca.crt - Certificate Authority cert
   * known_tokens.csv - tokens that entities (e.g. the kubelet) can use to talk to the apiserver
   * kubecfg.crt - Client certificate, public key
   * kubecfg.key - Client certificate, private key
   * server.cert - Server certificate, public key
   * server.key - Server certificate, private key
       -->
       接下来，您需要在每个节点上创建一个 `/srv/kubernetes/ `目录。这个目录包含：

   * basic_auth.csv  - 基本认证的用户名和密码
   * ca.crt - CA证书
   * known_tokens.csv - 用来和 apiserver 通信的令牌实体（例如 kubelet ）
   * kubecfg.crt - 客户端证书，公钥
   * kubecfg.key - 客户端证书，私钥
   * server.cert - 服务端证书，公钥
   * server.key - 服务端证书，私钥

<!--
The easiest way to create this directory, may be to copy it from the master node of a working cluster, or you can manually generate these files yourself.
-->
创建该目录最简单的方法可以是直接从一个工作正常的集群的 master 节点拷贝，或者您也可以手动生成它们。

<!--
### Starting the API Server
-->
### 启动 API Server

<!--
Once these files exist, copy the [kube-apiserver.yaml](/docs/admin/high-availability/kube-apiserver.yaml) into `/etc/kubernetes/manifests/` on each master node.
-->
一旦这些文件已经存在了，拷贝 [kube-apiserver.yaml](/docs/admin/high-availability/kube-apiserver.yaml) 到每个主节点的 `/etc/kubernetes/manifests/` 目录下。

<!--
The kubelet monitors this directory, and will automatically create an instance of the `kube-apiserver` container using the pod definition specified
in the file.
-->
kubelet 会监控这个目录，并且会按照文件里对 pod 的定义创建一个 `kube-apiserver` 容器。

<!--
### Load balancing
-->
### 负载均衡

<!--
At this point, you should have 3 apiservers all working correctly.  If you set up a network load balancer, you should
be able to access your cluster via that load balancer, and see traffic balancing between the apiserver instances.  Setting
up a load balancer will depend on the specifics of your platform, for example instructions for the Google Cloud
Platform can be found [here](https://cloud.google.com/compute/docs/load-balancing/)
-->
现在，您应该有3个全部正常工作的 apiserver 了。如果搭建了网络负载均衡器，您应该能够通过那个负载均衡器访问您的集群，并且看到负载在 apiserver 实例间分发。如何设置负载均衡器依赖于您的平台的实际情况，例如对于 Google Cloud Platform 的指导可以在[这里](https://cloud.google.com/compute/docs/load-balancing/)找到。

<!--
Note, if you are using authentication, you may need to regenerate your certificate to include the IP address of the balancer,
in addition to the IP addresses of the individual nodes.
-->
请注意，如果您使用了身份认证，可能需要重新生成证书，证书中除了需要有每个节点的 IP 地址外还需要额外包含负载均衡器的 IP 地址。

<!--
For pods that you deploy into the cluster, the `kubernetes` service/dns name should provide a load balanced endpoint for the master automatically.
-->
对于部署在集群中的 pod， `kubernetes` service/dns 名称应该自动的为 master 节点提供了负载均衡的 endpoint。

<!--
For external users of the API (e.g. the `kubectl` command line interface, continuous build pipelines, or other clients) you will want to configure
them to talk to the external load balancer's IP address.
-->
对于使用 API 访问的外部用户（如命令行运行的 `kubectl`，持续集成管道或其他客户端）您应该将他们配置成为访问外部负载均衡器的地址。

<!--
## Master elected components
-->
## 进行Master选举的组件

<!--
So far we have set up state storage, and we have set up the API server, but we haven't run anything that actually modifies
cluster state, such as the controller manager and scheduler.  To achieve this reliably, we only want to have one actor modifying state at a time, but we want replicated
instances of these actors, in case a machine dies.  To achieve this, we are going to use a lease-lock in the API to perform
master election.  We will use the `--leader-elect` flag for each scheduler and controller-manager, using a lease in the API will ensure that only 1 instance of the scheduler and controller-manager are running at once.
-->
到目前为止，我们已经搭建了状态存储，也搭建好了API server，但我们还没有运行任何真正改变集群状态的服务，比如 controller manager 和 scheduler。为了可靠的实现这个目标，我们希望在同一时间只有一个参与者在修改集群状态。但是我们希望复制这些参与者的实例以防某个机器宕机。要做到这一点，我们打算在 API 中使用一个 lease-lock 来执行 master 选举。我们会对每一个 scheduler 和 controller-manager 使用 `--leader-elect` 标志，从而在 API 中使用一个租约来保证同一时间只有一个 scheduler 和 controller-manager 的实例正在运行。

<!--
The scheduler and controller-manager can be configured to talk to the API server that is on the same node (i.e. 127.0.0.1), or it can be configured to communicate using the load balanced IP address of the API servers. Regardless of how they are configured, the scheduler and controller-manager will complete the leader election process mentioned above when using the `--leader-elect` flag.
-->
scheduler 和 controller-manager 可以配置为只和位于它们相同节点（即127.0.0.1）上的 API server 通信，也可以配置为使用 API server 的负载均衡器的 IP 地址。不管它们如何配置，当使用 `--leader-elect` 时 scheduler 和 controller-manager 都将完成上文提到的 leader 选举过程。

<!--
In case of a failure accessing the API server, the elected leader will not be able to renew the lease, causing a new leader to be elected. This is especially relevant when configuring the scheduler and controller-manager to access the API server via 127.0.0.1, and the API server on the same node is unavailable.
-->
为了防止访问 API server 失败，被选举出来的 leader 不能通过更新租约来选举一个新的 leader。当 scheduler 和 controller-manager 通过 127.0.0.1 访问 API server，而相同节点上的 API server 不可用时，这一点尤其重要。

<!--
### Installing configuration files
-->
### 安装配置文件

<!--
First, create empty log files on each node, so that Docker will mount the files not make new directories:

```shell
touch /var/log/kube-scheduler.log
touch /var/log/kube-controller-manager.log
```
-->
首先，在每个节点上创建空白日志文件，这样 Docker 就会挂载这些文件而不是创建一个新目录：

```shell
touch /var/log/kube-scheduler.log
touch /var/log/kube-controller-manager.log
```

<!--
Next, set up the descriptions of the scheduler and controller manager pods on each node.
by copying [kube-scheduler.yaml](/docs/admin/high-availability/kube-scheduler.yaml) and [kube-controller-manager.yaml](/docs/admin/high-availability/kube-controller-manager.yaml) into the `/etc/kubernetes/manifests/` directory.
-->
接下来，在每个节点上配置 scheduler 和 controller manager pod 的描述文件。拷贝 [kube-scheduler.yaml](/docs/admin/high-availability/kube-scheduler.yaml) 和 [kube-controller-manager.yaml](/docs/admin/high-availability/kube-controller-manager.yaml) 到 `/etc/kubernetes/manifests/` 目录。

<!--
## Conclusion
-->
## 结尾

<!--
At this point, you are done (yeah!) with the master components, but you still need to add worker nodes (boo!).
-->
此时，您已经完成了 master 组件的配置（耶！），但您还需要添加工作者节点（噗！）。

<!--
If you have an existing cluster, this is as simple as reconfiguring your kubelets to talk to the load-balanced endpoint, and
restarting the kubelets on each node.
-->
如果您有一个现成的集群，那么只需要在每个节点上简单的重新配置 kubelet 连接到负载均衡的 endpoint 并重启它们。

<!--
If you are turning up a fresh cluster, you will need to install the kubelet and kube-proxy on each worker node, and
set the `--apiserver` flag to your replicated endpoint.
-->
如果您搭建的是一个全新的集群，那么需要在每个 worker 节点上安装 kubelet 和 kube-proxy，并设置 `--apiserver` 指向多副本的 endpoint。