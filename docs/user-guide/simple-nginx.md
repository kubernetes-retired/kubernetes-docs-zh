---
---

好的，如果你已经开始了任何一个入门指南，并且启动了一个Kubernetes集群。那么接下来呢？ 这篇入门指南会帮助你在Kubernetes集群上运行第一个容器。

### 运行一个容器 (简单版)

从这时开始，我假设你已经根据其它入门指南安装了kubectl。

下面这行[`kubectl run`](/docs/user-guide/kubectl/kubectl_run) 命令会创建两个监听80端口的[nginx](https://registry.hub.docker.com/_/nginx/) [pods](/docs/user-guide/pods). 还会创建一名为`my-nginx`个[replication controller](/docs/user-guide/replication-controller),用来保证始终会有两个pod在运行。

```shell
kubectl run my-nginx --image=nginx --replicas=2 --port=80
```

一旦这些pod被创建好了， 你可以列出他们并查看他们的启动和运行。

```shell
kubectl get pods
```

你也能够看见replication controller被创建了：

```shell
kubectl get rc
```

如果要停止这两个被复制的容器，你可以通过停止replication controller:

```shell
kubectl stop rc my-nginx
```

### 将你的pod暴露给外网.

在特定的云平台上（例如Google Compute Engine），kubectl命令能够集成云端提供的API来给pod分配[公有IP地址](/docs/user-guide/services/#external-services)，可以通过以下命令来实现：

```shell
kubectl expose rc my-nginx --port=80 --type=LoadBalancer
```

这个命令会打印出被创建的service,以及映射到这些service的外部IP地址. 对外的IP地址根你实际运行环境有关。以Google Compute Engine为例，这些外部IP地址作为创建的service的一部分，可以使用如下指令查看到。

```shell
kubectl get services
```

为了访问你的nginx初始页面,你还不得不保证通过外部IP的通信是被允许的。那么就要通过让防火墙允许80端口通信才可以做到

### 接下来: 配置文件

鉴于大多数人愿意使用声明式的配置文件来创建或修改他们的应用程序，这里有另外一个文档给出了[简单介绍](/docs/user-guide/simple-yaml)。