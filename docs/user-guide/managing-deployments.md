---
---

你已经把应用部署完毕并且以Service的方式暴露出来了，那么接下来该做什么？Kubernetes提供了若干工具来帮助你管理部署工作，包括应用的水平扩容以及更新等。在所有这些功能里面，我们将要深入探讨的是[配置文件](/docs/user-guide/configuring-containers/#configuration-in-kubernetes)以及[Label](/docs/user-guide/deploying-applications/#labels)。

* TOC
{:toc}

## 编排资源的配置文件

许多应用都需要创建多种资源，比如一个Replication Controller以及一个Service。最简化的管理多种资源的方法就是把它们归集在同一个配置文件里（在YAML文件里用`---`分隔）。例如：

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

多种资源的创建方法和创建单一资源一样：

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

在只有两个资源的例子里，在命令行上同时指定它们的名字来删除也很简单，用资源/名字这种格式：

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

到目前为止的例子里，我们在多种资源里最多就使用了一个Label。但是在很多场景下我们需要使用多个Label来把不同的资源集合区分开。

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

通过指定Label，我们可以从不同的维度把资源切片：

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

## 啥 ？  
## Canary deployments

Another scenario where multiple labels are needed is to distinguish deployments of different releases or configurations of the same component. For example, it is common practice to deploy a *canary* of a new application release (specified via image tag) side by side with the previous release so that the new release can receive live production traffic before fully rolling it out. For instance, a new release of the guestbook frontend might carry the following labels:

```yaml
labels:
        app: guestbook
        tier: frontend
        track: canary
```

and the primary, stable release would have a different value of the `track` label, so that the sets of pods controlled by the two replication controllers would not overlap:

```yaml
labels:
        app: guestbook
        tier: frontend
        track: stable
```

The frontend service would span both sets of replicas by selecting the common subset of their labels, omitting the `track` label:

```yaml
selector:
     app: guestbook
     tier: frontend
```

## Updating labels

Sometimes existing pods and other resources need to be relabeled before creating new resources. This can be done with `kubectl label`. For example:

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

## Scaling your application

When load on your application grows or shrinks, it's easy to scale with `kubectl`. For instance, to increase the number of nginx replicas from 2 to 3, do:

```shell
$ kubectl scale rc my-nginx --replicas=3
scaled
$ kubectl get pods -lapp=nginx
NAME             READY     STATUS    RESTARTS   AGE
my-nginx-1jgkf   1/1       Running   0          3m
my-nginx-divi2   1/1       Running   0          1h
my-nginx-o0ef1   1/1       Running   0          1h
```

## Updating your application without a service outage

At some point, you'll eventually need to update your deployed application, typically by specifying a new image or image tag, as in the canary deployment scenario above. `kubectl` supports several update operations, each of which is applicable to different scenarios.

To update a service without an outage, `kubectl` supports what is called ['rolling update'?](/docs/user-guide/kubectl/kubectl_rolling-update), which updates one pod at a time, rather than taking down the entire service at the same time. See the [rolling update design document](https://github.com/kubernetes/kubernetes/blob/{{page.githubbranch}}/docs/design/simple-rolling-update.md) and the [example of rolling update](/docs/user-guide/update-demo/) for more information.

Let's say you were running version 1.7.9 of nginx:

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

To update to version 1.9.1, you can use [`kubectl rolling-update --image`](https://github.com/kubernetes/kubernetes/blob/{{page.githubbranch}}/docs/design/simple-rolling-update.md):

```shell
$ kubectl rolling-update my-nginx --image=nginx:1.9.1
Creating my-nginx-ccba8fbd8cc8160970f63f9a2696fc46
```

In another window, you can see that `kubectl` added a `deployment` label to the pods, whose value is a hash of the configuration, to distinguish the new pods from the old:

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

`kubectl rolling-update` reports progress as it progresses:

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

If you encounter a problem, you can stop the rolling update midway and revert to the previous version using `--rollback`:

```shell
$ kubectl kubectl rolling-update my-nginx  --image=nginx:1.9.1 --rollback
Found existing update in progress (my-nginx-ccba8fbd8cc8160970f63f9a2696fc46), resuming.
Found desired replicas.Continuing update with existing controller my-nginx.
Stopping my-nginx-02ca3e87d8685813dbe1f8c164a46f02 replicas: 1 -> 0
Update succeeded. Deleting my-nginx-ccba8fbd8cc8160970f63f9a2696fc46
my-nginx
```

This is one example where the immutability of containers is a huge asset.

If you need to update more than just the image (e.g., command arguments, environment variables), you can create a new replication controller, with a new name and distinguishing label value, such as:

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

and roll it out:

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

You can also run the [update demo](/docs/user-guide/update-demo/) to see a visual representation of the rolling update process.

## In-place updates of resources

Sometimes it's necessary to make narrow, non-disruptive updates to resources you've created. For instance, you might want to add an [annotation](/docs/user-guide/annotations) with a description of your object. That's easiest to do with `kubectl patch`:

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

The patch is specified using json.

For more significant changes, you can `get` the resource, edit it, and then `replace` the resource with the updated version:

```shell
$ kubectl get rc my-nginx-v4 -o yaml > /tmp/nginx.yaml
$ vi /tmp/nginx.yaml
$ kubectl replace -f /tmp/nginx.yaml
replicationcontrollers/my-nginx-v4
$ rm $TMP
```

The system ensures that you don't clobber changes made by other users or components by confirming that the `resourceVersion` doesn't differ from the version you edited. If you want to update regardless of other changes, remove the `resourceVersion` field when you edit the resource. However, if you do this, don't use your original configuration file as the source since additional fields most likely were set in the live state.

## Disruptive updates

In some cases, you may need to update resource fields that cannot be updated once initialized, or you may just want to make a recursive change immediately, such as to fix broken pods created by a replication controller. To change such fields, use `replace --force`, which deletes and re-creates the resource. In this case, you can simply modify your original configuration file:

```shell
$ kubectl replace -f ./nginx-rc.yaml --force
replicationcontrollers/my-nginx-v4
replicationcontrollers/my-nginx-v4
```

## What's next?

- [Learn about how to use `kubectl` for application introspection and debugging.](/docs/user-guide/introspection-and-debugging)
- [Tips and tricks when working with config](/docs/user-guide/config-best-practices
