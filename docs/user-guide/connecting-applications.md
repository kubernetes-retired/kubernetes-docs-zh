---
---

* TOC
{:toc}

# Kubernetes容器连接模型

既然已经有了一个可持续运行的多副本应用，现在就可以在网络中将它暴露出来了。在讨论Kubernetes的网络连接方式之前，很值得和Docker的常规网络连接方式做个对比。

Dokcer默认使用主机私有网络连接方式，所以只有在同一台物理机器上的容器之间才可以通信。为了让Docker容器可以跨节点通信，必须要给机器的IP地址分配端口号，这个端口之后会被用来转发或者路由给容器。很明显，这意味着容器要么要很小心地协调端口的使用，要么能动态地分配端口。

大规模地为多个开发者协调端口号不仅非常困难，而且会把无法把控的集群级别的问题暴露在用户面前。与Docker不同，Kubernetes默认假设Pod之间是可以直接通信的，而不管这些Pod分散在哪个主机上。而对于任意一个Pod来说，Kubernetes以Pod为单位分配一个集群私有的IP地址（cluster-private-IP address），所以我们不需要显示地创建容器之间的链接，也不需要映射容器的端口到主机的端口。这意味着Pod里的容器可以用localhost访问各自的端口，而且在没有NAT的情况下，集群中所有的Pod也互相可见。本文剩下的内容将会详细阐述如何在这样的网络模型中运行可靠的服务。

这个指南中用了一个简单的nginx服务来演示这个POC。同样的原理也在一个更完整的[Jenkins CI 应用](http://blog.kubernetes.io/2015/07/strong-simple-ssl-for-kubernetes.html)中体现了。

## 在集群中暴露Pod

在前面的例子中已经演示过，让我们把注意力集中在网络视角再来一次。创建一个nginx的`Pod`，请注意它定义了容器的端口：

```yaml
$ cat nginxrc.yaml
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

这让它变得可以从集群中的任一节点访问。检查一下Pod运行的节点：

```shell
$ kubectl create -f ./nginxrc.yaml
$ kubectl get pods -l app=nginx -o wide
my-nginx-6isf4   1/1       Running   0          2h        e2e-test-beeps-minion-93ly
my-nginx-t26zt   1/1       Running   0          2h        e2e-test-beeps-minion-93ly
```

检查Pod的IP地址：

```shell
$ kubectl get pods -l app=nginx -o json | grep podIP
                "podIP": "10.245.0.15",
                "podIP": "10.245.0.14",
```

在Kubernetes认可的网络模型下，你应该可以ssh到集群里的任何一个节点，而且用curl也能够访问这两个IP。要注意的是容器并*没有*用节点的80端口，也没用任何特殊的会把流量路由到Pod的NAT规则。这意味着你可以在同一个节点上用同样的`containerPort`配置运行多个nginx pod，而且通过IP就可以在其他Pod或者集群里的其他节点访问它们。和Docker类似，端口也可以在节点的网络中发布出来，但是在Kubernetes的这种网络模型下，这样的需求从根本上减少了。

如果你很好奇这个网络模型应该如何实现的话，可以在[我们如何做到的](/docs/admin/networking/#how-to-achieve-this)里读到具体细节。

## 创建Service

现在我们有了运行态的nginx，它们运行在一个水平的，集群范围的地址空间内。理论上，我们已经可以和这些Pod直接交互了，但是如果一个节点死掉了会发生什么？它里面的Pod也会死掉，然后Replication Controller会创建一个新的Pod，但是IP是不一样的。这就是Service解决的问题。

Kubernetes Service是对在集群中某处运行的一系列Pod的逻辑集合的抽象定义，这些Pod提供的功能是一样的。每个Service被创建的时候会被分配一个唯一的IP地址（也叫`clusterIP`）。这个地址和Service的生存周期紧密相关，而且只要Service活着就不会改变。Pod可以配置成和Service交互，并且知道和Service的通信会被自动地负载均衡到Service成员中的某个Pod。

可以用下面的yaml为两个nginx副本创建一个Service：

```yaml
$ cat nginxsvc.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginxsvc
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx
```

这个定义会创建一个Service，这个Service会把带有**app=nginx** Label的Pod的TCP 80端口暴露到Service的抽象端口（**targetPort**：是容器可以接收流量的端口，**port**：是Service的抽象端口，可以是除用来访问Service的端口之外的任何端口）。在Service定义中支持的所有的字段可以在[Service API 对象](http://kubernetes.io/v1.1/docs/api-reference/v1/definitions/#_v1_service)中查看。

查看你的Service：

```shell
$ kubectl get svc
NAME         LABELS        SELECTOR    IP(S)          PORT(S)
nginxsvc     app=nginx     app=nginx   10.0.116.146   80/TCP

```

在前面提到过，Service是由一组Pod支撑的。这些Pod通过**Endpoint**暴露出来。Service的Selector会被持续评估，并把结果发送给Endpoint对象（也叫做**nginxsvc**）。当一个Pod死了之后，它就会被自动地从Endpoint里面删掉，能够匹配Service的Selector的新Pod会被自动加到Endpoint里。检查Endpoint的时候也会看到IP和前一步里创建的Pod是一样的：

```shell
$ kubectl describe svc nginxsvc
Name:          nginxsvc
Namespace:     default
Labels:            app=nginx
Selector:      app=nginx
Type:          ClusterIP
IP:            10.0.116.146
Port:          <unnamed> 80/TCP
Endpoints:     10.245.0.14:80,10.245.0.15:80
Session Affinity:  None
No events.

$ kubectl get ep
NAME         ENDPOINTS
nginxsvc     10.245.0.14:80,10.245.0.15:80
```

现在你应该可以从集群里的任一节点上用curl命令访问**10.0.116.146:80**上的nginx Service了。要注意的是Service的IP完全是虚拟的，跟物理网络没有关系，它是不能够ping通的。如果你对它的工作原理有兴趣，可以去看看[service proxy](/docs/user-guide/services/#virtual-ips-and-service-proxies)。

## 访问Service

Kubernetes支持两种发现Service的主要模式：环境变量和DNS。环境变量在安装之后就可以直接使用，DNS模式需要[kube-dns 集群插件](http://releases.k8s.io/{{page.githubbranch}}/cluster/addons/dns/README.md)。

### 环境变量

当一个Pod在节点上运行时，kubelet会为每个活跃的Service添加一系列的环境变量。这引入了一个排序的问题。想要知道为什么，检查一下运行中的nginx Pod的环境：

```shell
$ kubectl exec my-nginx-6isf4 -- printenv | grep SERVICE
KUBERNETES_SERVICE_HOST=10.0.0.1
KUBERNETES_SERVICE_PORT=443
```

注意这里并没有出现你自己创建的Service，这是因为这些副本是在Service之前创建的。这样做的另一个缺点是，调度器也许会把两个Pod放到相同的机器上，如果机器出问题，整个Service就不工作了。正确的方式是把这两个Pod杀掉，然后等Replication Controller重建它们。由于此时的Service是在Pod副本之前就已经存在了，Kubernetes就会将同属于这个Service的Pod副本分布在不同的机器上，这给予了Pod调度器级别的Service扩散能力（只要所有的节点的容量是一样的），而且环境变量也是正确的：

```shell
$ kubectl scale rc my-nginx --replicas=0; kubectl scale rc my-nginx --replicas=2;
$ kubectl get pods -l app=nginx -o wide
NAME             READY   STATUS     RESTARTS   AGE   NODE
my-nginx-5j8ok   1/1     Running   	0         2m    node1
my-nginx-90vaf   1/1     Running   0          2m    node2

$ kubectl exec my-nginx-5j8ok -- printenv | grep SERVICE
KUBERNETES_SERVICE_PORT=443
NGINXSVC_SERVICE_HOST=10.0.116.146
KUBERNETES_SERVICE_HOST=10.0.0.1
NGINXSVC_SERVICE_PORT=80
```

### DNS

Kubernetes提供了一个带DNS插件的Service，它可以自动的用skydns给其他Service分配DNS域名。你可以查看一下它是否已在你的cluster中运行：

```shell
$ kubectl get services kube-dns --namespace=kube-system
NAME       CLUSTER_IP      EXTERNAL_IP   PORT(S)         SELECTOR           AGE
kube-dns   10.179.240.10   <none>        53/UDP,53/TCP   k8s-app=kube-dns   8d
```

如果它不在运行，你可以[启动它](http://releases.k8s.io/{{page.githubbranch}}/cluster/addons/dns/README.md#how-do-i-configure-it)。本文剩余部分假定你已经有了一个已分配固定IP的Service（nginxsvc），一个DNS服务器，并且已经给Service的IP分配了DNS域名。这样，你用标准方法（比如gethostbyname）就可以从集群中的任何Pod访问到这个Service了。让我们再创建一个Pod测试一下：

```yaml
$ cat curlpod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: curlpod
spec:
  containers:
  - image: radial/busyboxplus:curl
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: curlcontainer
  restartPolicy: Always
```

再查看一下nginx的Service：

```shell
$ kubectl create -f ./curlpod.yaml
default/curlpod
$ kubectl get pods curlpod
NAME      READY     STATUS    RESTARTS   AGE
curlpod   1/1       Running   0          18s

$ kubectl exec curlpod -- nslookup nginxsvc
Server:    10.0.0.10
Address 1: 10.0.0.10
Name:      nginxsvc
Address 1: 10.0.116.146
```

## 让Service更加安全

到目前为止，我们还只是在集群内部访问nginx服务。在把这个Service暴露到Internet之前，你一定想要确认通信渠道是安全的。为了达到这个目的，你需要：

* 自签名的https证书（除非你有认证的证书）
* 配置好用这个证书的nginx服务
* 让Pod能访问证书的[Secret](/docs/user-guide/secrets)

可以从[nginx https示例](https://github.com/kubernetes/kubernetes/tree/{{page.githubbranch}}/examples/https-nginx/)获取所有的信息，一个简单的例子：

```shell
$ make keys secret KEY=/tmp/nginx.key CERT=/tmp/nginx.crt SECRET=/tmp/secret.json
$ kubectl create -f /tmp/secret.json
secrets/nginxsecret
$ kubectl get secrets
NAME                  TYPE                                  DATA
default-token-il9rc   kubernetes.io/service-account-token   1
nginxsecret           Opaque                                2
```

现在修改nginx副本，让它用Secret里面的证书启动一个HTTPS服务。它的Service要同时暴露80和443两个端口：

```yaml
$ cat nginx-app.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginxsvc
  labels:
    app: nginx
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    protocol: TCP
    name: https
  selector:
    app: nginx
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
      - name: secret-volume
        secret:
          secretName: nginxsecret
      containers:
      - name: nginxhttps
        image: bprashanth/nginxhttps:1.0
        ports:
        - containerPort: 443
        - containerPort: 80
        volumeMounts:
        - mountPath: /etc/nginx/ssl
          name: secret-volume
```

nginx-app中值得关注的点是：

- 在一个文件中同时包含了Replication Controller和Service的定义
- [nginx 服务器](https://github.com/kubernetes/kubernetes/tree/{{page.githubbranch}}/examples/https-nginx/default.conf)在80端口提供HTTP服务，在443端口提供HTTPS服务，nginx的Service暴露了这两个端口。
- 每个容器都可以通过挂载在/etc/nginx/ssl上的Volume访问这些Key。这是在nginx服务器启动之前就设定好的。

```shell
$ kubectl delete rc,svc -l app=nginx; kubectl create -f ./nginx-app.yaml
replicationcontrollers/my-nginx
services/nginxsvc
services/nginxsvc
replicationcontrollers/my-nginx
```

现在你就可以从任何节点访问nginx服务器了。

```shell
$ kubectl get pods -o json | grep -i podip
    "podIP": "10.1.0.80",
node $ curl -k https://10.1.0.80
...
<h1>Welcome to nginx!</h1>
```

也许你注意到了在上一步的curl命令里带了`-k`参数，这是因为在证书生成的时候我们对要运行nginx服务的Pod一无所知，因此需要让curl忽略CNAME不匹配导致的错误。通过创建一个Service，我们在Service查询期间把证书里使用的CNAME和Pod实际使用的DNS域名链接在一起。让我们用一个Pod测试一下（为了简单起见，我们重用了同一个Secret，这个Pod只需要nginx.crt就能访问Service）：

```shell
$ cat curlpod.yaml
vapiVersion: v1
kind: ReplicationController
metadata:
  name: curlrc
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: curlpod
    spec:
      volumes:
      - name: secret-volume
        secret:
          secretName: nginxsecret
      containers:
      - name: curlpod
        command:
        - sh
        - -c
        - while true; do sleep 1; done
        image: radial/busyboxplus:curl
        volumeMounts:
        - mountPath: /etc/nginx/ssl
          name: secret-volume

$ kubectl create -f ./curlpod.yaml
$ kubectl get pods
NAME             READY     STATUS    RESTARTS   AGE
curlpod          1/1       Running   0          2m
my-nginx-7006w   1/1       Running   0          24m

$ kubectl exec curlpod -- curl https://nginxsvc --cacert /etc/nginx/ssl/nginx.crt
...
<title>Welcome to nginx!</title>
...
```

## 暴露Service

因为你的应用的一些部分，也许你想要把Service暴露到一个外部的IP地址。Kubernetes支持两种方式：NodePort以及LoadBalancer。在前面小节里创建的Service已经使用了`NodePort`，因此如果节点有公网IP，你的nginx HTTPS副本已经准备好接收来自Internet的流量了。

```shell
$ kubectl get svc nginxsvc -o json | grep -i nodeport -C 5
            {
                "name": "http",
                "protocol": "TCP",
                "port": 80,
                "targetPort": 80,
                "nodePort": 32188
            },
            {
                "name": "https",
                "protocol": "TCP",
                "port": 443,
                "targetPort": 443,
                "nodePort": 30645
            }

$ kubectl get nodes -o json | grep ExternalIP -C 2
                    {
                        "type": "ExternalIP",
                        "address": "104.197.63.17"
                    }
--
                    },
                    {
                        "type": "ExternalIP",
                        "address": "104.154.89.170"
                    }
$ curl https://104.197.63.17:30645 -k
...
<h1>Welcome to nginx!</h1>
```

接下来，让我们试试用云平台内置的负载均衡器重新创建Service，只要把nginx-app.yaml里面Service的`Type`字段从`NodePort`修改为`LoadBalancer`就可以：

```shell
$ kubectl delete rc, svc -l app=nginx
$ kubectl create -f ./nginx-app.yaml
$ kubectl get svc nginxsvc
NAME      CLUSTER_IP       EXTERNAL_IP       PORT(S)                SELECTOR     AGE
nginxsvc  10.179.252.126   162.222.184.144   80/TCP,81/TCP,82/TCP   run=nginx2   13m

$ curl https://162.22.184.144 -k
...
<title>Welcome to nginx!</title>
```

`EXTERNAL_IP`栏位的IP地址就可以在Internet上被访问了。`CLUSTER_IP`只能在集群或者私有云网络内部访问。

## 下一节

[在生产环境帮你更可靠地运行容器的Kubernetes功能](/docs/user-guide/production-pods)
