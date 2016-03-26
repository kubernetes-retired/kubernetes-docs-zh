---
---

在前面的章节里，我们了解了如何用[`kubectl run`](/docs/user-guide/quick-start)快速部署一个简单的多副本应用以及如何用Pod配置并生成单次运行的容器（[configuring-containers.md](/docs/user-guide/configuring-containers)）。本文，我们将使用基于配置的方法来部署一个持续运行的多副本应用。

* TOC
{:toc}

## 用配置文件生成一组容器副本

Kubernetes用[`Replication Controller`](/docs/user-guide/replication-controller)创建并管理多副本的容器集合（实际上是多副本的[Pod](/docs/user-guide/pods)）。

`Replication Controller`简单地确保在任一时间里都有特定数量的pod副本在运行。如果运行的太多，它会杀掉一些；如果运行的太少，它会启动一些。这和Google Compute Engine的[Instance Group Manager](https://cloud.google.com/compute/docs/instance-groups/manager/)以及AWS的[Auto-scaling Group](http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/AutoScalingGroup)（不包含弹性策略）类似。

在[快速开始](/docs/user-guide/quick-start)章节里用`kubctl run`创建的用来运行Nginx的`Replication Controller`可以用下面的YAML描述：

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

和指定一个单独的Pod相比，不同的是设置了这里的`kind`字段为`ReplicationController`，设定了需要的副本数量以及把Pod的定义放到了template字段下面。Pod的名字不需要显示指定，因为它们是根据`Replication Controller`的名字生成的。要查看支持的字段列表，可以查看[Replication Controller API object](http://kubernetes.io/v1.1/docs/api-reference/v1/definitions/#_v1_replicationcontroller)。

和创建pods一样，也可以用`create`命令来创建这个replication controller：

```shell
$ kubectl create -f ./nginx-rc.yaml
replicationcontrollers/my-nginx
```

Replication Controller会重建删除的或者因不明原因终止的（比如节点失败）Pod，这和直接创建的Pod的情况是不一样。基于这样的考量，对于一个需要持续运行的应用，即便你的应用只需要一个单独的pod，我们也推荐使用Replication Controller。对于单独的Pod，在配置文件里可以省略`replicas`这个字段，因为不设置的时候默认就只有一个副本。

## 查看Replication Controller的状态

可以用`get`命令查看你创建的Replication Controller：

```shell
$ kubectl get rc
CONTROLLER   CONTAINER(S)   IMAGE(S)   SELECTOR    REPLICAS
my-nginx     nginx          nginx      app=nginx   2
```

这说明你的Controller会确保有两个nginx的副本。

和直接创建的Pod一样，也可以用`get`命令查看这些副本：

```shell
$ kubectl get pods
NAME             READY     STATUS    RESTARTS   AGE
my-nginx-065jq   1/1       Running   0          51s
my-nginx-buaiq   1/1       Running   0          51s
```

## 删除replication controllers

如果想要结束你的应用并且删除Repication Controller。和在[快速开始](/docs/user-guide/quick-start)里一样，用下面的命令：

```shell
$ kubectl delete rc my-nginx
replicationcontrollers/my-nginx
```

这个操作默认会把由Replication Controller管理的Pod一起删除。如果Pod的数量比较多，这个操作要花一些时间才能完成。如果想要Pod继续运行，不被删掉，可以在delete的时候指定参数`--cascade=false`。

如果在删除Replication Controller之前想要删除Pod，Pod只是被重建了，因为Replication Controller会再启动新的Pod，确保Pod的数量。

## Labels

Kubernetes使用自定义的键值对（称为[*Label*](/docs/user-guide/labels)）分类资源集合，例如Pod和Replication Controller。在前面的例子里，Pod的模板里只设定了一个单独的Label，键是`app`，值为`nginx`。所有被创建的Pod都带有这个Label，可以用带`-L`参数的命令查看：

```shell
$ kubectl get pods -L app
NAME             READY     STATUS    RESTARTS   AGE       APP
my-nginx-afv12   0/1       Running   0          3s        nginx
my-nginx-lg99z   0/1       Running   0          3s        nginx
```

Pod模板中的Label默认也会被拷贝到Replication Controller的Label字段中。

Kubernetes中所有的资源都支持Label：

```shell
$ kubectl get rc my-nginx -L app
CONTROLLER   CONTAINER(S)   IMAGE(S)   SELECTOR    REPLICAS   APP
my-nginx     nginx          nginx      app=nginx   2          nginx
```

更重要的是，Pod模板的Label会被用来创建[`Selector`](/docs/user-guide/labels/#label-selectors)，这个`Selector`会匹配所有带这些Label的Pod。用`kubectl get`的[go语言模板输出格式](/docs/user-guide/kubectl/kubectl_get)就可以看到这个字段：

```shell
$ kubectl get rc my-nginx -o template --template="{{.spec.selector}}"
map[app:nginx]
```

如果你想要在Pod模板里指定Label，但是又不想使用这些Label进行筛选，可以显示指定`Selector`来解决，不过需要确保`Selector`能够匹配由Pod模板创建出来的Pod的Label，并且不能匹配由其他Replication Controller创建的Pod。对于后者，最直接最保险的方法是给Replication Controller分配一个独特的Label，并且在Pod模板和Selector里都进行指定。
## 下一节

[向用户和客户端暴露应用，以及如何把应用串联起来](/docs/user-guide/connecting-applications)。
