---
---

* TOC
{:toc}

# Kubernetes容器连接模型

既然已经有了一个可持续运行的多副本应用，现在就可以在网络中将它暴露出来了。在讨论Kubernetes的网络连接方式之前，很值得和Docker的常规网络连接方式做个对比。

Dokcer默认使用主机私有网络连接方式，所以只有在同一台物理机器上的容器之前才可以通信。为了让Docker容器可以跨节点通信，必须要给机器的IP地址分配端口号，这个端口之后会被用来转发或者路由给容器。很明显，这意味着容器要么很小心地协调使用端口，要么有动态分配地端口。

在一定的规模下，为多个开发者协调端口号非常困难。这也会把集群级别的问题暴露给用户，这是在用户的控制之外的。Kubernetes假定Pod之间是可以通信的，不管它们落到哪个主机上。我们给每个Pod指定集群私有的IP地址（cluster-private-IP address），所以不需要显示地创建Pod之间的链接，也不需要映射容器的端口到主机的端口。这意味着Pod里的容器可以在本机（localhost）上访问各自的端口，而且在没有NAT的情况下，集群中所有的Pod也可以互相可见的。本文剩下的内容将会详细阐述如何在这样的网络模型中运行可靠的服务。

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

你应该可以ssh到集群里的任何一个节点，而且用curl也能够访问这两个IP。要注意的是容器并*没有*用节点的80端口，也没用任何特殊的会把流量路由到Pod的NAT规则。这意味着你可以在同一个节点上用同样的`containerPort`配置运行多个nginx pod，而且通过IP就可以在其他Pod或者集群里的其他节点访问它们。和Docker类似，端口也可以在节点的网络中发布出来，但是在Kubernetes的这种网络模型下，这样的需求从根本上减少了。

如果你很好奇，可以在[我们如何做到的](/docs/admin/networking/#how-to-achieve-this)里读到更多细节。

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

现在你应该可以从集群里的任一节点上用curl命令访问**10.0.116.146:80**上的nginx Service了。要注意的是Service的IP完全是虚拟的，跟物理网络没有关系，如果你对它的工作原理有兴趣，可以去看看[service proxy](/docs/user-guide/services/#virtual-ips-and-service-proxies)。

## 访问Service

Kubernetes支持两种发现Service的主要模式：环境变量和DNS。环境变量在安装之后就可以直接使用，DNS模式需要[kube-dns 集群插件](http://releases.k8s.io/{{page.githubbranch}}/cluster/addons/dns/README.md)。

### 环境变量

当一个Pod在节点上运行时，kubelet会为每个活跃的Service添加一系列的环境变量。这引入了一个排序的问题。想要知道为什么，检查一下运行中的nginx Pod的环境：

```shell
$ kubectl exec my-nginx-6isf4 -- printenv | grep SERVICE
KUBERNETES_SERVICE_HOST=10.0.0.1
KUBERNETES_SERVICE_PORT=443
```

注意这里并没有提到Service，这是因为这些副本是在Service之前创建的。这样做的另一个缺点是，调度器也许会把两个Pod放到相同的机器上，如果机器出问题，整个Service就不工作了。正确的方式是把这两个Pod杀掉，然后等Replication Controller重建它们。现在Service是在Pod副本之前存在了，这给予了Pod调度器级别的Service扩散能力（只要所有的节点的容量是一样的），而且环境变量也是正确的：

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

Kubernetes offers a DNS cluster addon Service that uses skydns to automatically assign dns names to other Services. You can check if it's running on your cluster:

```shell
$ kubectl get services kube-dns --namespace=kube-system
NAME       CLUSTER_IP      EXTERNAL_IP   PORT(S)         SELECTOR           AGE
kube-dns   10.179.240.10   <none>        53/UDP,53/TCP   k8s-app=kube-dns   8d
```

If it isn't running, you can [enable it](http://releases.k8s.io/{{page.githubbranch}}/cluster/addons/dns/README.md#how-do-i-configure-it). The rest of this section will assume you have a Service with a long lived IP (nginxsvc), and a dns server that has assigned a name to that IP (the kube-dns cluster addon), so you can talk to the Service from any pod in your cluster using standard methods (e.g. gethostbyname). Let's create another pod to test this:

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

And perform a lookup of the nginx Service

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

## Securing the Service

Till now we have only accessed the nginx server from within the cluster. Before exposing the Service to the internet, you want to make sure the communication channel is secure. For this, you will need:

* Self signed certificates for https (unless you already have an identity certificate)
* An nginx server configured to use the certificates
* A [secret](/docs/user-guide/secrets) that makes the certificates accessible to pods

You can acquire all these from the [nginx https example](https://github.com/kubernetes/kubernetes/tree/{{page.githubbranch}}/examples/https-nginx/), in short:

```shell
$ make keys secret KEY=/tmp/nginx.key CERT=/tmp/nginx.crt SECRET=/tmp/secret.json
$ kubectl create -f /tmp/secret.json
secrets/nginxsecret
$ kubectl get secrets
NAME                  TYPE                                  DATA
default-token-il9rc   kubernetes.io/service-account-token   1
nginxsecret           Opaque                                2
```

Now modify your nginx replicas to start a https server using the certificate in the secret, and the Service, to expose both ports (80 and 443):

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

Noteworthy points about the nginx-app manifest:

- It contains both rc and service specification in the same file
- The [nginx server](https://github.com/kubernetes/kubernetes/tree/{{page.githubbranch}}/examples/https-nginx/default.conf) serves http traffic on port 80 and https traffic on 443, and nginx Service exposes both ports.
- Each container has access to the keys through a volume mounted at /etc/nginx/ssl. This is setup *before* the nginx server is started.

```shell
$ kubectl delete rc,svc -l app=nginx; kubectl create -f ./nginx-app.yaml
replicationcontrollers/my-nginx
services/nginxsvc
services/nginxsvc
replicationcontrollers/my-nginx
```

At this point you can reach the nginx server from any node.

```shell
$ kubectl get pods -o json | grep -i podip
    "podIP": "10.1.0.80",
node $ curl -k https://10.1.0.80
...
<h1>Welcome to nginx!</h1>
```

Note how we supplied the `-k` parameter to curl in the last step, this is because we don't know anything about the pods running nginx at certificate generation time,
so we have to tell curl to ignore the CName mismatch. By creating a Service we linked the CName used in the certificate with the actual DNS name used by pods during Service lookup.
Lets test this from a pod (the same secret is being reused for simplicity, the pod only needs nginx.crt to access the Service):

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

## Exposing the Service

For some parts of your applications you may want to expose a Service onto an external IP address. Kubernetes supports two ways of doing this: NodePorts and LoadBalancers. The Service created in the last section already used `NodePort`, so your nginx https replica is ready to serve traffic on the internet if your node has a public IP.

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

Lets now recreate the Service to use a cloud load balancer, just change the `Type` of Service in the nginx-app.yaml from `NodePort` to `LoadBalancer`:

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

The IP address in the `EXTERNAL_IP` column is the one that is available on the public internet.  The `CLUSTER_IP` is only available inside your
cluster/private cloud network.

## What's next?

[Learn about more Kubernetes features that will help you run containers reliably in production.](/docs/user-guide/production-pods)
