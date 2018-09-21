---
title:  监控Kubernetes中的资源利用
date: 2015-05-12
slug: resource-usage-monitoring-kubernetes
url: /blog/2015/05/Resource-Usage-Monitoring-Kubernetes
---

<!--
---
title:  Resource Usage Monitoring in Kubernetes
date: 2015-05-12
slug: resource-usage-monitoring-kubernetes
url: /blog/2015/05/Resource-Usage-Monitoring-Kubernetes
---
-->

<!--
Understanding how an application behaves when deployed is crucial to scaling the application and providing a reliable service. In a Kubernetes cluster, application performance can be examined at many different levels: containers, [pods](http://kubernetes.io/docs/user-guide/pods), [services](http://kubernetes.io/docs/user-guide/services), and whole clusters. As part of Kubernetes we want to provide users with detailed resource usage information about their running applications at all these levels. This will give users deep insights into how their applications are performing and where possible application bottlenecks may be found. In comes [Heapster](https://github.com/kubernetes/heapster), a project meant to provide a base monitoring platform on Kubernetes.  
-->

了解应用程序在部署时的行为方式对于扩展应用程序和提供可靠服务至关重要。 在 Kubernetes 集群中，可以在许多不同级别检查应用程序性能：容器，[pods](http://kubernetes.io/docs/user-guide/pods)，[services](http://kubernetes.io/docs/user-guide/services) 和整个集群。作为 Kubernetes 的一部分，我们希望为用户提供他们运行程序所有级别的详细资源使用信息。这将使用户深入了解他们的应用程序如何执行以及可能找到应用程序瓶颈。如下的 [Heapster](https://github.com/kubernetes/heapster) 是一个在 Kubernetes 中提供基础监控平台的项目。


<!--
**Overview**  


Heapster is a cluster-wide aggregator of monitoring and event data. It currently supports Kubernetes natively and works on all Kubernetes setups. Heapster runs as a pod in the cluster, similar to how any Kubernetes application would run. The Heapster pod discovers all nodes in the cluster and queries usage information from the nodes’ [Kubelets](https://github.com/kubernetes/kubernetes/blob/master/DESIGN.md#kubelet), the on-machine Kubernetes agent. The Kubelet itself fetches the data from [cAdvisor](https://github.com/google/cadvisor). Heapster groups the information by pod along with the relevant labels. This data is then pushed to a configurable backend for storage and visualization. Currently supported backends include [InfluxDB](http://influxdb.com/) (with [Grafana](http://grafana.org/) for visualization), [Google Cloud Monitoring](https://cloud.google.com/monitoring/) and many others described in more details here. The overall architecture of the service can be seen below:  
-->

**概述**

Heapster 是一个集群范围的监控和事件数据聚合器。它目前原生支持 Kubernetes ，并适用于所有 Kubernetes 计划。Heapster 作为集群中的 pod 运行，类似于任何 Kubernetes 应用程序的运行方式。Heapster pod 发现集群中的所有节点，并通过节点的 [Kubelets](https://github.com/kubernetes/kubernetes/blob/master/DESIGN.md#kubelet) (Kubernetes 在物理机上的代理) 查询使用情况。 Kubelet 从每个机器上的 [cAdvisor](https://github.com/google/cadvisor) 代理获取数据。Heapster 按照每个 Pod 和其相关标签进行分组。然后将该数据推送到可配置的后端进行存储和可视化。目前支持的后端包括[InfluxDB](http://influxdb.com/) (使用 [Grafana](http://grafana.org/)进行可视化)，[Google Cloud Monitoring](https://cloud.google.com/monitoring/) 和这里描述的其它应用。该服务的整体架构如下所示：

[![](https://2.bp.blogspot.com/-6Bu15356Zqk/V4mGINP8eOI/AAAAAAAAAmk/-RwvkJUt4rY2cmjqYFBmRo25FQQPRb27ACEw/s640/monitoring-architecture.png)](https://2.bp.blogspot.com/-6Bu15356Zqk/V4mGINP8eOI/AAAAAAAAAmk/-RwvkJUt4rY2cmjqYFBmRo25FQQPRb27ACEw/s1600/monitoring-architecture.png)

<!--
Let’s look at some of the other components in more detail.
-->

让我们更详细地看一些其他组件。

<!--
**cAdvisor**

cAdvisor is an open source container resource usage and performance analysis agent. It is purpose built for containers and supports Docker containers natively. In Kubernetes, cadvisor is integrated into the Kubelet binary. cAdvisor auto-discovers all containers in the machine and collects CPU, memory, filesystem, and network usage statistics. cAdvisor also provides the overall machine usage by analyzing the ‘root’? container on the machine.

On most Kubernetes clusters, cAdvisor exposes a simple UI for on-machine containers on port 4194. Here is a snapshot of part of cAdvisor’s UI that shows the overall machine usage:  
-->

**cAdvisor**

cAdvisor 是一个开源的容器资源使用和性能分析代理。它专为容器而生，本身支持 Docker 容器。在 Kubernetes 中，cadvisor 被整合到 Kubelet 二进制文件中。cAdvisor 自动发现机器中的所有容器，并收集 CPU 、内存、文件系统和网络的使用情况的统计信息。cAdvisor 也提供整体机器使用情况，是通过分析 'root' 获取的吗？ 机器上的容器。
 
在大多数 Kubernetes 集群上，cAdvisor 在4194端口上为机器上的容器创建了一个简单的 UI 界面。以下是 cAdvisor UI 部分的快照，显示了整个机器的使用情况：


[![](https://3.bp.blogspot.com/-V5KAfomW7Cg/V4mGH6OTKSI/AAAAAAAAAmo/EZHcG0afrs0606eTDMCryT6j6SoNzu3PgCEw/s400/cadvisor.png)](https://3.bp.blogspot.com/-V5KAfomW7Cg/V4mGH6OTKSI/AAAAAAAAAmo/EZHcG0afrs0606eTDMCryT6j6SoNzu3PgCEw/s1600/cadvisor.png)

<!--
**Kubelet**  

The Kubelet acts as a bridge between the Kubernetes master and the nodes. It manages the pods and containers running on a machine. Kubelet translates each pod into its constituent containers and fetches individual container usage statistics from cAdvisor. It then exposes the aggregated pod resource usage statistics via a REST API.

**STORAGE BACKENDS**

**InfluxDB and Grafana**
-->

**Kubelet**  

Kubelet 作为 Kubernetes master 和节点之间的桥梁。它管理在机器上运行的 pod 和容器。Kubelet 将每个 pod 转换为其组成的容器，并从 cAdvisor 获取每个容器的使用情况统计信息。然后，它通过 REST API 导出聚合的 pod 资源使用情况统计信息。

**STORAGE BACKENDS**

**InfluxDB and Grafana**

<!--
A Grafana setup with InfluxDB is a very popular combination for monitoring in the open source world. InfluxDB exposes an easy to use API to write and fetch time series data. Heapster is setup to use this storage backend by default on most Kubernetes clusters. A detailed setup guide can be found [here](https://github.com/kubernetes/heapster/blob/master/docs/influxdb.md). InfluxDB and Grafana run in Pods. The pod exposes itself as a Kubernetes service which is how Heapster discovers it.
-->

在开源的世界中，Grafana 和 InfluxDB 是非常流行的组合。InfluxDB 提供了一个易于使用的 API 来写入和获取时间序列数据。大多数 Kubernetes 集群中，Heapster 默认使用它作为存储后端。详细的设置指南可以在[这里](https://github.com/kubernetes/heapster/blob/master/docs/influxdb.md)找到。InfluxDB 和 Grafana 在 Pods 中运行。pod 通过 Kubernetes service 显示自己，Heapster 通过这种方式发现它。

<!--
The Grafana container serves Grafana’s UI which provides an easy to configure dashboard interface. The default dashboard for Kubernetes contains an example dashboard that monitors resource usage of the cluster and the pods inside of it. This dashboard can easily be customized and expanded. Take a look at the storage schema for InfluxDB [here](https://github.com/kubernetes/heapster/blob/master/docs/storage-schema.md#metrics).

Here is a video showing how to monitor a Kubernetes cluster using heapster, InfluxDB and Grafana:
-->

Grafana 容器提供可以方便配置仪表板的 Grafana UI。Kubernetes 的默认仪表板包含一个示例仪表板，用于监视集群及其内部的 pod 该仪表板可以轻松定制和扩展。[这里](https://github.com/kubernetes/heapster/blob/master/docs/storage-schema.md#metrics) 有 InfluxDB 的存储结构

这有一个视频，展示了如何使用 heapster、InfluxDB 和 Grafana 监控 Kubernetes 集群：


 [![](https://img.youtube.com/vi/SZgqjMrxo3g/0.jpg)](https://www.youtube.com/watch?SZgqjMrxo3g)

<!--
Here is a snapshot of the default Kubernetes Grafana dashboard that shows the CPU and Memory usage of the entire cluster, individual pods and containers:
-->

以下是默认 Kubernetes Grafana 仪表板的快照，其中显示了整个群集、各个 pod 和容器的 CPU 和内存使用情况：

[![](https://1.bp.blogspot.com/-lHMeU_4UnAk/V4mGHyrWkBI/AAAAAAAAAms/SvnncgJ7ieAduBqQzpI86oaboIkAKEpEQCEw/s640/influx.png)](https://1.bp.blogspot.com/-lHMeU_4UnAk/V4mGHyrWkBI/AAAAAAAAAms/SvnncgJ7ieAduBqQzpI86oaboIkAKEpEQCEw/s1600/influx.png)

**Google Cloud Monitoring**

<!--
Google Cloud Monitoring is a hosted monitoring service that allows you to visualize and alert on important metrics in your application. Heapster can be setup to automatically push all collected metrics to Google Cloud Monitoring. These metrics are then available in the [Cloud Monitoring Console](https://app.google.stackdriver.com/). This storage backend is the easiest to setup and maintain. The monitoring console allows you to easily create and customize dashboards using the exported data.

Here is a video showing how to setup and run a Google Cloud Monitoring backed Heapster:
"https://youtube.com/embed/xSMNR2fcoLs"
Here is a snapshot of the a Google Cloud Monitoring dashboard showing cluster-wide resource usage.
-->

Google Cloud Monitoring 是一种托管监控服务，可让您对应用程序中的重要指标进行可视化和警报。可以将 Heapster 设置为自动将所有收集的指标推送到 Google Cloud Monitoring。然后，可以在 [Cloud Monitoring Console](https://app.google.stackdriver.com/) 中使用这些指标。此存储后端是最容易设置和维护的。监控控制台允许您使用导出的数据轻松创建和自定义仪表板。

以下是显示如何设置和运行支持 Heapster 的 Google Cloud Monitoring  的视频：

"https://youtube.com/embed/xSMNR2fcoLs"

以下是 Google Cloud Monitoring 仪表板的快照，其中显示了群集范围的资源使用情况。

[![](https://2.bp.blogspot.com/-F2j3kYn3IoA/V4mGH3M-0gI/AAAAAAAAAmg/aoml93zPeKsKbTX1tN5sTtRRTw7dAKsxwCEw/s640/gcm.png)](https://2.bp.blogspot.com/-F2j3kYn3IoA/V4mGH3M-0gI/AAAAAAAAAmg/aoml93zPeKsKbTX1tN5sTtRRTw7dAKsxwCEw/s1600/gcm.png)

**Try it out!**

<!--
Now that you’ve learned a bit about Heapster, feel free to try it out on your own clusters! The [Heapster repository](https://github.com/kubernetes/heapster) is available on GitHub. It contains detailed instructions to setup Heapster and its storage backends. Heapster runs by default on most Kubernetes clusters, so you may already have it! Feedback is always welcome. Please let us know if you run into any issues via the troubleshooting channels.
-->

既然您已经了解了 Heapster ，可以随意在自己的集群上进行尝试！ [Heapster repository](https://github.com/kubernetes/heapster) 可在 GitHub 上获得。它包含设置 Heapster 及其存储后端的详细说明。默认情况下，Heapster 在大多数 Kubernetes 集群上运行，因此您可能已经拥有它！随时欢迎反馈。如果您通过故障排除渠道遇到任何问题，请告知我们。


_-- Vishnu Kannan and Victor Marmol, Google Software Engineers_
