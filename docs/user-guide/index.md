---
---

* TOC
{:toc}

用户指南是为所有想在已经运行着的Kubernetes集群上运行程序或者服务准备的。Kubernetes集群的设置和管理可以参见[Cluster Admin Guide](/docs/admin/)。而[Developer Guide](https://github.com/kubernetes/kubernetes/tree/{{page.githubbranch}}/docs/devel/) 是为所有想要用Kubernetes API来编写代码的工程师或者想直接为Kubernetes贡献代码的人准备的。

请确保你已经阅读完毕[运行用户指南中的示例的必读事项](/docs/user-guide/prereqs).

## 快速指南

1. [Kubernetes 101](/docs/user-guide/walkthrough/)
1. [Kubernetes 201](/docs/user-guide/walkthrough/k8s201)

## 详细指南

如果你对Kubernetes的熟悉程度还不够，我们建议您按如下的顺序阅读指南：

1. [快速开始：启动并开放一个应用](/docs/user-guide/quick-start)
1. [容器的配置与启动：常用容器参数的配置](/docs/user-guide/configuring-containers)
1. [运行中应用的持续部署](/docs/user-guide/deploying-applications)
1. [应用的连接：将应用开放给客户端和用户](/docs/user-guide/connecting-applications)
1. [与生产环境中的容器打交道](/docs/user-guide/production-pods)
1. [部署的管理](/docs/user-guide/managing-deployments)
1. [应用的检视与调试](/docs/user-guide/introspection-and-debugging)
    1. [使用Kubernetes的Web用户界面](/docs/user-guide/ui)
    1. [日志](/docs/user-guide/logging)
    1. [监控](/docs/user-guide/monitoring)
    1. [通过`exec`切入容器](/docs/user-guide/getting-into-containers)
    1. [通过代理连接容器](/docs/user-guide/connecting-to-applications-proxy)
    1. [通过端口转发连接容器](/docs/user-guide/connecting-to-applications-port-forward)

## 概要

Kubernetes是一个能在集群中跨多主机管理容器化应用的的开源系统。Kubernetes意在让那些容器化的应用以及基于微服务的应用的部署变得简单但功能强大。

Kubernetes提供了诸多机制用来进行应用部署，调度，更新，维护和伸缩。Kubernetes的一个关键特性是它能主动的管理容器来保证集群的状态持续地符合用户的期望状态。运维人员能够启动微服务，然后让调度器来为它找到合适的安置点。我们也想不断的改善工具和用户体验，让他们能通过如金丝雀部署等模式来放出(roll-out)应用。

Kubernetes支持[Docker](http://www.docker.io)和[Rocket](https://coreos.com/blog/rocket/)容器, 在未来还会加入对其他的容器镜像格式和容器runtime的支持。

尽管Kubernetes目前集中关注持续运行的无状态的（如web服务器,内存中的对象缓存）和云原生有状态的应用（如NoSQL数据库），在很快的将来，也会支持所有的其他的在生产集群环境能常见到的workload类型，例如批处理，流式处理，和传统的数据库。

Kubernetes中，所有的容器都运行在pod中，一个pod来容纳一个单独的容器，或者多个合作的容器。在后一种情况，pod中的容器被保证放置在同一个机器上，可以共享资源。一个pod也能包含零个或者更多的的volume，volume即目录,可以是对于一个容器私有,或者在pod中的容器间共享。对于用户每个创建的pod，系统会找到一个正常运转并且有足够的容量的机器，然后将相应的容器在那里启动。如果一个容器出现故障，它会被Kubernetes的node agent自动重启，这个node agent被称作Kubelet。但是如果pod或者他的机器出故障，它不会被自动转移或者重启，除非用户也定义了一个replication controller，我们马上就会讲到它。

用户可以自己创建并管理Pod，但是Kubernetes极大的简化了系统管理，它能让用户指派两个常见的跟pod相关的活动：基于相同的Pod配置部署多个pod副本，和在一个pod或者它所在的机器发生故障的时候创建取代性的Pod。在Kubernetes的API对象中,用来管理这些行为是Replication Controller，它用模板的形式定义了pod，然后系统根据模板实例化出一些pod（特别是由用户指定）。Pod的副本集合可以共同组成一整个应用，一个微服务，或者一个多层应用的某一层。一旦Pod创建好，系统会持续地监控他们的健康状态，和它们运行时所在的机器的健康状况。如果一个Pod因为软件问题或者所在机器故障出现问题，Replication控制器会自动在健康的机器上创建一个新的Pod，来保证Pod的集合处于一个期望的冗余水平。一个或者多个应用的多个Pod能共享一个机器。注意对于单个没有副本的Pod,如果用户想在这个Pod或者其运行所在的机器出现故障的时候能重新创建,那也需要用到Replication Controller。


能够方便地引用一个pod集合是一个常见需要用到的功能。例如，需要一个修改操作只对一个在限制范围内的集合生效，或者限制一个查询状态操作的Pod集合范围。作为一个常见的机制，Kubernetes绝大多是的API对象都允许用户添加任意的key-value键值对，被称作Label，并且可以让用户用一系列的Label选择器（对label的键值对查询）来对API操作目标进行限制。每一个资源也有一个字符串类型的键值可以被外部的工具来存储或者获取任意的元数据，被称作Annotations。

Kubernetes支持一种独特的网络模型。Kubernetes鼓励用扁平的地址空间，并且不会动态地分配端口，而是采用让用户可以选择任意合适自己的端口。为了实现这点，它给每一个pod分配了一个ip地址。

现代的Internel应用通常由层级的微服务构建而来，例如一组前端和分布式的内存内键值存储交互，和有多个副本的存储服务交互。为了构建这种架构，Kubernetes提供了service的抽象，其提供了一个稳定的IP地址和DNS名字，来对应一组动态的pod，例如一组构成一个微服务的pod。这个pod组是通过label选择器来定义的，因为可以指定任何的pod组。当一个运行在Kubernetes pod里的容器连接到这个地址时，这个连接会被本地的代理转发（称作kube proxy）。该代理运行在来源机器上。转发的目的地是一个相应的后端容器，确切的后端是通过轮询的策略进行选择，以均衡负载。kube proxy也会追踪后端的pod组的动态变化，如当pod被位于新机器上的新的pod取代的时候，而服务的IP和DNS名字不用改变。


每一个Kubernetes中的资源，如Pod，都通过一个URI来被识别，并且有一个UID。URI中一个重要的属性是，对象的类型（如：pod）,对象的名字，和对象的Namespace（命名空间）。对于一个特定的对象类型，每一个名字在其命名空间都是独一无二的。在一个对象的名字没有带着命名空间的形式给出，那就是默认的命名空间。UID在时间和空间的范围都是唯一的。


## 概念指南


[**Cluster**](/docs/admin/)
: Cluster是一组物理的或者虚拟的机器，也包含其他运行Kubernetes所需的基础设施资源

[**Node**](/docs/admin/node)
: 一个Node是一个物理的或者虚拟的运行Kubernetes的机器，Pod就是在Node上面进行调度的

[**Pod**](/docs/user-guide/pods)
: 一个Pod是放置在一起的一组容器和卷

[**Label**](/docs/user-guide/labels)
: Label指的是一个可以附加在资源上的键值对，例如可以附加在Pod上来传达一个用户定义的具有辨识性的属性。Label可以用来组织资源，或者选取资源的特定子集。

[**Selector**](/docs/user-guide/labels/#label-selectors)
: Selector是一个匹配Label的表达式，目的是用来辨识相关的资源，如一个负载均衡的服务的目标Pod是哪些。

[**Replication Controller**](/docs/user-guide/replication-controller)
: Replication Controller是用来确保在同一时间有指定数量的Pod副本正在运行。它不但让伸缩变得简单，同时让在机器重启或者因为其他原因出故障的时候对一个Pod进行重建。

[**Service**](/docs/user-guide/services)
: Service定义了一组Pod和一个访问这组Pod的方式，例如一个稳定的IP地址和相应的DNS名称。

[**Volume**](/docs/user-guide/volumes)
: 卷是一个目录，里面很可能包含数据，可以被容器作为自己文件系统的一部分进行访问。Kubernetes的卷基于[Docker Volumes](https://docs.docker.com/userguide/dockervolumes/)构建而来，并且添加了volume directory和/或device的设置。

[**Secret**](/docs/user-guide/secrets)
: Secret里面保存敏感的数据，例如用于认证的令牌，可以在请求的时候为容器所用。

[**Name**](/docs/user-guide/identifiers)
: 一个用户或者客户端提供的资源名称。

[**Namespace**](/docs/user-guide/namespaces)
: 命名空间就像一个资源名称的前缀。命名空间能通过如防止不相关的项目组发生命名冲突的措施,帮助不同的项目，项目组，或者客户来共享集群。

[**Annotation**](/docs/user-guide/annotations)
: 可以用来保存更大（相比较于label）的键值对，并且可能包含可读性低的数据，目的是用来保存非辨认性目的的数据，特别是那些由工具和系统扩展操作的数据。annotation的值无法用来进行有效地过滤。

## 深入阅读

* API资源
  * [使用Resouce](/docs/user-guide/working-with-resources)

* Pods和容器
  * [Pod的生命周期和重启策略](/docs/user-guide/pod-states)
  * [生命周期中的钩子](/docs/user-guide/container-environment)
  * [计算资源，如CPU和内存](/docs/user-guide/compute-resources)
  * [指定命令和请求能力](/docs/user-guide/containers)
  * [Downward API: 从Pod中访问系统配置](/docs/user-guide/downward-api)
  * [镜像和注册表](/docs/user-guide/images)
  * [从docker-cli到kubectl的迁移](/docs/user-guide/docker-cli-to-kubectl)
  * [配置的相关提示和小技巧](/docs/user-guide/config-best-practices)
  * [向指定的Node分配Pod](/docs/user-guide/node-selection/)
  * [在一组正在运行的Pod执行滚动更新](/docs/user-guide/update-demo/)
