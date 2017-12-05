---
title: 本地开发和调试 services
cn-approvers:
- pigletfly
---
<!--
---
title: Developing and debugging services locally
---
-->
{% capture overview %}
<!--
Kubernetes applications usually consist of multiple, separate services, each running in its own container. Developing and debugging these services on a remote Kubernetes cluster can be cumbersome, requiring you to [get a shell on a running container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/) and running your tools inside the remote shell.
-->
Kubernetes 应用程序通常由多个独立的 services 组成， 每个 service 都运行在自己的容器里。在一个远程的 Kubernetes 集群中开发和调试这些 services 可能会比较笨重，需要你[在一个运行的容器中获取 shell](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)，然后在这个远程 shell 中运行你的工具。

<!--
`telepresence` is a tool to ease the process of developing and debugging services locally, while proxying the service to a remote Kubernetes cluster. Using `telepresence` allows you to use custom tools, such as a debugger and IDE, for a local service and provides the service full access to ConfigMap, secrets, and the services running on the remote cluster.

This document describes using `telepresence` to develop and debug services running on a remote cluster locally.
-->

`telepresence` 是一个让本地开发和调试 services 更加容易的工具，当代理 service 到远程 Kubernetes 集群时。`telepresence` 可以允许你对于一个本地 service 使用自定义工具，比如一个调试器和 IDE，并且赋予这个 service 对运行在远程集群上的 ConfigMap，secrets，和 services 的全部访问权限。

这个文档描述了如何使用 `telepresence` 对运行在远程集群中的 services 在本地开发和调试。

{% endcapture %}

{% capture prerequisites %}
<!--
* Kubernetes cluster is installed
* `kubectl` is configured to communicate with the cluster
* [Telepresence](https://www.telepresence.io/reference/install) is installed
-->

* Kubernetes 集群已经安装
* `kubectl` 已经配置好可以与集群通信
* [Telepresence](https://www.telepresence.io/reference/install)  已经安装

{% endcapture %}

{% capture steps %}

<!--
## Getting a shell on a remote cluster

Open a terminal and run `telepresence` with no arguments to get a `telepresence` shell. This shell runs locally, giving you full access to your local filesystem.

The `telepresence` shell can be used in a variety of ways. For example, write a shell script on your laptop, and run it directly from the shell in real time. You can do this on a remote shell as well, but you might not be able to use your preferred code editor, and the script is deleted when the container is terminated.

Enter `exit` to quit and close the shell.
-->

## 获取一个在远程集群上的 shell

打开一个终端然后运行 `telepresence` 不带任何参数，获取一个 `telepresence` shell。这个 shell 在本地运行，使你可以完全访问本地文件系统。

`telepresence` shell 可以以各种方式使用。比如，在你的笔记本电脑上写一个 shell 脚本，然后可以在 `telepresence` shell 中直接实时运行。你也可以在一个远程 shell 中这么操作，但是你可能不能使用你喜欢的代码编辑器，并且当容器终止的时候脚本会被删除。

输入 `exit` 退出和关闭 shell。

<!--
## Developing or debugging an existing service

When developing an application on Kubernetes, you typically program or debug a single service. The service might require access to other services for testing and debugging. One option is to use the continuous deployment pipeline, but even the fastest deployment pipeline introduces a delay in the program or debug cycle.

Use the `--swap-deployment` option to swap an existing deployment with the Telepresence proxy. Swapping allows you to run a service locally and connect to the remote Kubernetes cluster. The services in the remote cluster can now access the locally running instance.

To run telepresence with `--swap-deployment`, enter:

`telepresence --swap-deployment $DEPLOYMENT_NAME`

where $DEPLOYMENT_NAME is the name of your existing deployment.

Running this command spawns a shell. In the shell, start your service. You can then make edits to the source code locally, save, and see the changes take effect immediately. You can also run your service in a debugger, or any other local development tool.

-->

## 开发或调试现有的 service

当在 Kubernetes 上开发一个应用程序的时候，一般你只会编写或者调试一个 service。这个 service 可能需要访问其他 services 用于测试和调试。一种选择是使用持续部署 pipeline，但是即使最快的部署 pipeline 也可能导致在编码或者调试周期中延期。

使用 `--swap-deployment` 参数用于通过 Telepresence 代理交换一个已经存在的 deployment。交换允许你在本地运行一个 service，然后连接到远程的 Kubernetes 集群。现在在远程集群中的 services 可以访问本地运行的实例。

去运行 telepresence 带上 `--swap-deployment`，输入：

`telepresence --swap-deployment $DEPLOYMENT_NAME`

$DEPLOYMENT_NAME 是你的已经存在的 deployment 的名字。

运行这个命令会产生一个 shell。 在这个 shell 里，启动你的 service。你可以在本地编辑源码，保存，然后查看立即生效的变化。你也可以在一个调试器或者任何其他的本地开发工具中运行你的 service 。


{% endcapture %}

{% capture whatsnext %}
<!--
If you're interested in a hands-on tutorial, check out [this tutorial](https://cloud.google.com/community/tutorials/developing-services-with-k8s) that walks through locally developing the Guestbook application on Google Container Engine.

Telepresence has [numerous proxying options](https://www.telepresence.io/reference/methods), depending on your situation.

For further reading, visit the [Telepresence website](https://www.telepresence.io).
-->

如果你对动手教程感兴趣，查看 [这个教程] (https://cloud.google.com/community/tutorials/developing-services-with-k8s)，它从头到尾教指导你在 Google Container Engine 上本地开发 Guestbook 应用。

Telepresence 有 [许多代理选项](https://www.telepresence.io/reference/methods)，取决于你的情况。

想要进一步阅读，访问 [Telepresence website](https://www.telepresence.io)。


{% endcapture %}

{% include templates/task.md %}
