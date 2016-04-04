---
---

你已经把应用部署完毕并且以Service的方式暴露出来了，那么接下来该做什么？Kubernetes提供了若干工具来帮助你管理部署工作，包括应用的水平扩容以及更新等。在所有这些功能里面，我们将要深入探讨的是[配置文件](/docs/user-guide/configuring-containers/#configuration-in-kubernetes)以及[Label](/docs/user-guide/deploying-applications/#labels)。

* TOC
{:toc}

## 编排资源的配置文件

许多应用都需要创建多个资源，比如一个Replication Controller以及一个Service。最简化的管理多个资源的方法就是把它们归集在同一个配置文件里（在YAML文件里用`---`分隔）。例如：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-svc
  labels:
    app: nginx
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: nginx
---
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

多个资源的创建方法和创建单一资源一样：

```shell
$ kubectl create -f ./nginx-app.yaml
services/my-nginx-svc
replicationcontrollers/my-nginx
```

资源会按照在文件中出现的顺序被创建。因此，最好把Service放在前面，因为这样会确保当Replication Controller创建Pod的时候，调度器可以把Pod和Service关联起来，并扩散到不同的机器中。

`kubectl create` 同样支持多个 `-f` 参数:

```shell
$ kubectl create -f ./nginx-svc.yaml -f ./nginx-rc.yaml
```

除了指定多个文件之外，也可以指定一个目录：

```shell
$ kubectl create -f ./nginx/
```

`kubectl` 会读取所有后缀为`.yaml`，`.yml`和`.json`的文件。

比较推荐的做法是把同一个微服务或者子应用相关的资源放在同一个配置文件里，然后把同一个应用相关的所有配置文件归集到同一个目录中。如果你的应用的各个子系统通过DNS互相绑定，你就可以一次性把这些组件全部部署起来。

除了文件，URL也可以成为配置源。这对于部署GitHub上面的配置文件非常方便：

```shell
$ kubectl create -f https://raw.githubusercontent.com/GoogleCloudPlatform/kubernetes/master/docs/user-guide/replication.yaml
replicationcontrollers/nginx
```

## kubectl批量操作

`kubectl`可以批量操作的不仅仅是创建资源，`kubectl`还可以从配置文件中提取资源名称来进行其他操作，特别是用来删除从配置文件中创建出来的资源：

```shell
$ kubectl delete -f ./nginx/
replicationcontrollers/my-nginx
services/my-nginx-svc
```

在只有两个资源的例子里，在命令行上同时指定它们的名字来删除也很简单，用『资源/名字』这种格式：

```shell
$ kubectl delete replicationcontrollers/my-nginx services/my-nginx-svc
```

对于大量的资源，可以使用Label来筛选资源。通过`-l`参数来指定Selector：

```shell
$ kubectl delete all -lapp=nginx
replicationcontrollers/my-nginx
services/my-nginx-svc
```

因为`kubectl`输出的资源名和它接受的语法是一样的，所以用`$()`或者`xargs`就可以很简单地把一些操作串联起来：

```shell
$ kubectl get $(kubectl create -f ./nginx/ | grep my-nginx)
CONTROLLER   CONTAINER(S)   IMAGE(S)   SELECTOR    REPLICAS
my-nginx     nginx          nginx      app=nginx   2
NAME           LABELS      SELECTOR    IP(S)          PORT(S)
my-nginx-svc   app=nginx   app=nginx   10.0.152.174   80/TCP
```

## 高效地使用Label

到目前为止的例子里，我们在多个资源里最多就使用了一个Label。但是在很多场景下我们需要使用多个Label来把不同的资源集合区分开。

例如，不同的应用会给`app`这个Label不同的值，但是对于有多个子系统的应用，比如[guest example](https://github.com/kubernetes/kubernetes/tree/{{page.githubbranch}}/examples/guestbook/)，还需要把各个子系统区分开。应用的前端也许会有这样的Label：

```yaml
labels:
        app: guestbook
        tier: frontend
```

然后Redis的Master和Slave的`tier` Label是和前端不同的，也许Redis还会有一个`role` Label：

```yaml
labels:
        app: guestbook
        tier: backend
        role: master
```

以及

```yaml
labels:
        app: guestbook
        tier: backend
        role: slave
```

通过指定Label，我们可以从不同的维度区分资源：

```shell
$ kubectl create -f ./guestbook-fe.yaml -f ./redis-master.yaml -f ./redis-slave.yaml
replicationcontrollers/guestbook-fe
replicationcontrollers/guestbook-redis-master
replicationcontrollers/guestbook-redis-slave
$ kubectl get pods -Lapp -Ltier -Lrole
NAME                           READY     STATUS    RESTARTS   AGE       APP         TIER       ROLE
guestbook-fe-4nlpb             1/1       Running   0          1m        guestbook   frontend   <n/a>
guestbook-fe-ght6d             1/1       Running   0          1m        guestbook   frontend   <n/a>
guestbook-fe-jpy62             1/1       Running   0          1m        guestbook   frontend   <n/a>
guestbook-redis-master-5pg3b   1/1       Running   0          1m        guestbook   backend    master
guestbook-redis-slave-2q2yf    1/1       Running   0          1m        guestbook   backend    slave
guestbook-redis-slave-qgazl    1/1       Running   0          1m        guestbook   backend    slave
my-nginx-divi2                 1/1       Running   0          29m       nginx       <n/a>      <n/a>
my-nginx-o0ef1                 1/1       Running   0          29m       nginx       <n/a>      <n/a>
$ kubectl get pods -lapp=guestbook,role=slave
NAME                          READY     STATUS    RESTARTS   AGE
guestbook-redis-slave-2q2yf   1/1       Running   0          3m
guestbook-redis-slave-qgazl   1/1       Running   0          3m
```

## 灰度发布

需要用多个Label的另外一种场景是用来区分应用的不同版本或者同一个组件的不同配置的部署。比如，在生产环境发布应用的新版本时会先部署小部分的新版本应用，这样新版本和老版本会并存一段时间，而且新版本可以收到真实的流量，以便进行应用的线上认证。当确认新版本没有问题后，就会把老版本完全替换掉。请看下面的例子，guestbook Frontend的新版本也许会有这些Label：

```yaml
labels:
        app: guestbook
        tier: frontend
        track: canary
```

而主要的Frontend稳定版本的`track` Label的值是不一样的，因此这两组Pod分别被不同的Replication Controller管理，不会出现互相覆盖的情况：

```yaml
labels:
        app: guestbook
        tier: frontend
        track: stable
```

Frontend Service只选择它们Label之间的公共部分，忽略`track` Label，这样Service就可以同时覆盖到两个副本集：

```yaml
selector:
     app: guestbook
     tier: frontend
```

## 更新Label

在创建新的资源之前，一些已经创建的Pod以及其他资源或许需要重新标记Label。这可以用`kubectl label`命令解决，比如：

```shell
$ kubectl label pods -lapp=nginx tier=fe
NAME                READY     STATUS    RESTARTS   AGE
my-nginx-v4-9gw19   1/1       Running   0          14m
NAME                READY     STATUS    RESTARTS   AGE
my-nginx-v4-hayza   1/1       Running   0          13m
NAME                READY     STATUS    RESTARTS   AGE
my-nginx-v4-mde6m   1/1       Running   0          17m
NAME                READY     STATUS    RESTARTS   AGE
my-nginx-v4-sh6m8   1/1       Running   0          18m
NAME                READY     STATUS    RESTARTS   AGE
my-nginx-v4-wfof4   1/1       Running   0          16m
$ kubectl get pods -lapp=nginx -Ltier
NAME                READY     STATUS    RESTARTS   AGE       TIER
my-nginx-v4-9gw19   1/1       Running   0          15m       fe
my-nginx-v4-hayza   1/1       Running   0          14m       fe
my-nginx-v4-mde6m   1/1       Running   0          18m       fe
my-nginx-v4-sh6m8   1/1       Running   0          19m       fe
my-nginx-v4-wfof4   1/1       Running   0          16m       fe
```

## 应用弹性扩容

当你的应用的负载增加或者减轻时，用`kubectl`可以很简单地进行弹性扩容。比如，把nginx的副本数从2增加到3：

```shell
$ kubectl scale rc my-nginx --replicas=3
scaled
$ kubectl get pods -lapp=nginx
NAME             READY     STATUS    RESTARTS   AGE
my-nginx-1jgkf   1/1       Running   0          3m
my-nginx-divi2   1/1       Running   0          1h
my-nginx-o0ef1   1/1       Running   0          1h
```

## 热部署的方式更新在线应用

在某些时间点上，你最终会需要更新已部署的应用，通常会是指定一个新的镜像或者更新到镜像新的tag，就像上面的灰度发布场景一样。`kubectl`支持好几种更新操作，分别适用于不同的场景。

在服务不中断的情况下更新，`kubectl`支持所谓的['rolling update'](/docs/user-guide/kubectl/kubectl_rolling-update)？，一次只更新一个Pod，而不是同时把整个服务都停掉。查看[rolling update设计文档](https://github.com/kubernetes/kubernetes/blob/{{page.githubbranch}}/docs/design/simple-rolling-update.md)以及[rolling update例子](/docs/user-guide/update-demo/)获取更多信息。

假设你现在在运行的是nginx的1.7.9版本：

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx
spec:
  replicas: 5
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

要想更新到1.9.1版本，你可以用[`kubectl rolling-update --image`](https://github.com/kubernetes/kubernetes/blob/{{page.githubbranch}}/docs/design/simple-rolling-update.md)：

```shell
$ kubectl rolling-update my-nginx --image=nginx:1.9.1
Creating my-nginx-ccba8fbd8cc8160970f63f9a2696fc46
```

在另一个窗口，你可以看到`kubectl`给要更新的Pod增加了`deployment` Label。为了把新的Pod和老的区分开，这个`deployment` Label的值是配置文件的哈希值：

```shell
$ kubectl get pods -lapp=nginx -Ldeployment
NAME                                              READY     STATUS    RESTARTS   AGE       DEPLOYMENT
my-nginx-1jgkf                                    1/1       Running   0          1h        2d1d7a8f682934a254002b56404b813e
my-nginx-ccba8fbd8cc8160970f63f9a2696fc46-k156z   1/1       Running   0          1m        ccba8fbd8cc8160970f63f9a2696fc46
my-nginx-ccba8fbd8cc8160970f63f9a2696fc46-v95yh   1/1       Running   0          35s       ccba8fbd8cc8160970f63f9a2696fc46
my-nginx-divi2                                    1/1       Running   0          2h        2d1d7a8f682934a254002b56404b813e
my-nginx-o0ef1                                    1/1       Running   0          2h        2d1d7a8f682934a254002b56404b813e
my-nginx-q6all                                    1/1       Running   0          8m        2d1d7a8f682934a254002b56404b813e
```

`kubectl rolling-update`会汇报更新的进度：

```shell
Updating my-nginx replicas: 4, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 1
At end of loop: my-nginx replicas: 4, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 1
At beginning of loop: my-nginx replicas: 3, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 2
Updating my-nginx replicas: 3, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 2
At end of loop: my-nginx replicas: 3, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 2
At beginning of loop: my-nginx replicas: 2, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 3
Updating my-nginx replicas: 2, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 3
At end of loop: my-nginx replicas: 2, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 3
At beginning of loop: my-nginx replicas: 1, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 4
Updating my-nginx replicas: 1, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 4
At end of loop: my-nginx replicas: 1, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 4
At beginning of loop: my-nginx replicas: 0, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 5
Updating my-nginx replicas: 0, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 5
At end of loop: my-nginx replicas: 0, my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 replicas: 5
Update succeeded. Deleting old controller: my-nginx
Renaming my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 to my-nginx
my-nginx
```

如果更新遇到问题，你可以在中途停掉更新，并用`--rollback`回滚到之前的版本：

```shell
$ kubectl kubectl rolling-update my-nginx  --image=nginx:1.9.1 --rollback
Found existing update in progress (my-nginx-ccba8fbd8cc8160970f63f9a2696fc46), resuming.
Found desired replicas.Continuing update with existing controller my-nginx.
Stopping my-nginx-02ca3e87d8685813dbe1f8c164a46f02 replicas: 1 -> 0
Update succeeded. Deleting my-nginx-ccba8fbd8cc8160970f63f9a2696fc46
my-nginx
```

这是一个展示容器在不变性上巨大优势的例子。

如果你想要更新的不仅仅是镜像（例如命令的参数，环境变量等），你可以创建一个新的Replication Controller，起一个新的名字和有区别的Label值，比如：

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx-v4
spec:
  replicas: 5
  selector:
    app: nginx
    deployment: v4
  template:
    metadata:
      labels:
        app: nginx
        deployment: v4
    spec:
      containers:
      - name: nginx
        image: nginx:1.9.2
        args: ['nginx'?,'?-T'?]
        ports:
        - containerPort: 80
```

然后开始更新：

```shell
$ kubectl rolling-update my-nginx -f ./nginx-rc.yaml
Creating my-nginx-v4
At beginning of loop: my-nginx replicas: 4, my-nginx-v4 replicas: 1
Updating my-nginx replicas: 4, my-nginx-v4 replicas: 1
At end of loop: my-nginx replicas: 4, my-nginx-v4 replicas: 1
At beginning of loop: my-nginx replicas: 3, my-nginx-v4 replicas: 2
Updating my-nginx replicas: 3, my-nginx-v4 replicas: 2
At end of loop: my-nginx replicas: 3, my-nginx-v4 replicas: 2
At beginning of loop: my-nginx replicas: 2, my-nginx-v4 replicas: 3
Updating my-nginx replicas: 2, my-nginx-v4 replicas: 3
At end of loop: my-nginx replicas: 2, my-nginx-v4 replicas: 3
At beginning of loop: my-nginx replicas: 1, my-nginx-v4 replicas: 4
Updating my-nginx replicas: 1, my-nginx-v4 replicas: 4
At end of loop: my-nginx replicas: 1, my-nginx-v4 replicas: 4
At beginning of loop: my-nginx replicas: 0, my-nginx-v4 replicas: 5
Updating my-nginx replicas: 0, my-nginx-v4 replicas: 5
At end of loop: my-nginx replicas: 0, my-nginx-v4 replicas: 5
Update succeeded. Deleting my-nginx
my-nginx-v4
```

你也可以运行[更新演示](/docs/user-guide/update-demo/)来看rolling update过程的可视化展示。

## 原地更新资源

有些时候很有必要对已创建的资源进行局部的无干扰的更新。比如你可能会想要添加一个[Annotation](/docs/user-guide/annotations)描述一下这个资源。最简单的办法就是运行`kubectl patch`：

```shell
$ kubectl patch rc my-nginx-v4 -p '{"metadata": {"annotations": {"description": "my frontend running nginx"}}}' 
my-nginx-v4
$ kubectl get rc my-nginx-v4 -o yaml
apiVersion: v1
kind: ReplicationController
metadata:
  annotations:
    description: my frontend running nginx
...
```

这个补丁是使用JSON描述的。

对于内容变化很多的情况，你可以用`get`命令获取资源描述，然后编辑它，再用`replace`命令把它替换成更新后的版本：

```shell
$ kubectl get rc my-nginx-v4 -o yaml > /tmp/nginx.yaml
$ vi /tmp/nginx.yaml
$ kubectl replace -f /tmp/nginx.yaml
replicationcontrollers/my-nginx-v4
$ rm $TMP
```

（通过get命令导出的内容中包含一个resourceVersion的字段），系统通过检测resourceVersion字段与当前部署版本是否一致来确保你的更改不会意外覆盖掉其他人已经执行的更改。如果你无论如何都要使用自己的更改，而不在意其他人已经更改的内容，可以（在执行replace命令前）去除掉resourceVersion这个字段。然而，如果你决定这样做，请不要用原始那份配置文件作为源（而要用get命令返回的那份），因为它在运行状态下会设置一些额外的字段。

## 破坏性更新

在有些情况下，你会需要更新资源的一些字段，但是这些字段是在初始化之后就无法更新的。又或许你想要立即进行一些嵌套更新，比如修复Replication Controller创建失败的Pod。要想改变这些字段，用`replace --force`命令，这个操作会删掉然后重新创建这些资源。这种情况下，你可以在原始的配置文件上直接修改（而不必用get命令取回来的那个），然后：

```shell
$ kubectl replace -f ./nginx-rc.yaml --force
replicationcontrollers/my-nginx-v4
replicationcontrollers/my-nginx-v4
```

## 下一节

- [用`kubectl`回顾以及调试应用](/docs/user-guide/introspection-and-debugging)
- [使用配置的提示与技巧](/docs/user-guide/config-best-practices)
