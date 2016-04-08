---
---

`secret`的对象是用来保存敏感信息，比如密码，OAuth令牌，和ssh的key。把这些信息放在`secret`里面比放在一个`pod`定义中或者docker镜像中更加安全。更多详情请查看[Secrets设计文档](https://github.com/kubernetes/kubernetes/blob/{{page.githubbranch}}/docs/design/secrets.md) 。

* TOC
{:toc}

## Secret概览

Secret是一个包含少量敏感信息的对象，如密码，一个token或者一个key。这些信息也可以放在Pod的定义中或者镜像中；把它放在Secret对象中能更有效地对它的使用进行控制，并且减少不经意的泄露风险。

用户可以创建Secret，系统自身也会创建一些Secret。

要使用Secret，一个pod需要对该Secret进行引用。

Pod可以以两种方式来使用Secret：或者作为文件，通过pod中一个或多个容器挂载的[volume](/docs/user-guide/volumes)进行传递，或者在kubelet为pod拉取镜像的时候使用。

### Service Account将自动创建并附加包含API认证信息的Secret

Kubernetes会自动创建包含API认证信息的Secret，并且它会自动修改Pod来使用这种类型的Secret。

API认证信息的自动创建和使用可以在想要的时候被禁用或者修改。然而，如果你只需要安全的访问apiserver，这是一种推荐的使用方式。

[Service Account](/docs/user-guide/service-accounts)文档有更多的关于Service Account工作原理的信息

### 手动创建一个Secret

这是一个简单secret的例子，格式是yaml：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: dmFsdWUtMg0K
  username: dmFsdWUtMQ0K
```

data字段是一个map。它的键必须符合[`DNS_SUBDOMAIN`](https://github.com/kubernetes/kubernetes/blob/{{page.githubbranch}}/docs/design/identifiers.md)的要求，除此之外，开头的点（`.`）是允许的。它的值是使用base64编码的任意数据。上面例子中username和password的值，在base64编码之前，分别是`value-1`和`value-2`，后面跟着回车和换行符。

使用[`kubectl create`](/docs/user-guide/kubectl/kubectl_create)来创建secret。

一旦secret创建好，你需要修改pod来指定它使用这个secret。

### 手动指定一个要挂载到Pod的Secret

下面这个例子中的pod挂载了一个volume中的secret：

```json
{
 "apiVersion": "v1",
 "kind": "Pod",
  "metadata": {
    "name": "mypod",
    "namespace": "myns"
  },
  "spec": {
    "containers": [{
      "name": "mypod",
      "image": "redis",
      "volumeMounts": [{
        "name": "foo",
        "mountPath": "/etc/foo",
        "readOnly": true
      }]
    }],
    "volumes": [{
      "name": "foo",
      "secret": {
        "secretName": "mysecret"
      }
    }]
  }
}
```


每一个你想要使用的secret需要指定自己的spec.volumes字段。

如果pod中有多个容器，那么容器需要自己的`volumeMounts`块，但是一个secret只需要一个`spec.volumes`。

你可以把多个文件打包进一个secret中，或者使用多secret，你可以选择任意一种方便的方式。

[这里](/docs/user-guide/secrets/)有另外一个创建一个secret和在volume中使用该secret的例子。

### 手动指定一个imagePullSecret

imagePullSecrets的使用可以在[images文档](/docs/user-guide/images/#specifying-imagepullsecrets-on-a-pod)中找到。

### 自动附加imagePullSecret

你可以手动创建一个imagePullSecret，然后在serviceAccount中引用它。任何用该serviceAccount创建的pod，或者那些默认使用该Service Account的pod，它们的imagePullSecret字段都会设置成该service account的该字段。更多该流程的详细细节，请查看这里[这里](/docs/user-guide/service-accounts/#adding-imagepullsecrets-to-a-service-account)。


### 手动创建的Secret的自动挂载

我们计划扩展service account的行为，让手动创建的secret（如包含一个访问github账号的令牌的secret）可以根据它们的service account自动附加到该pod上。

*这目前还没有被实现。请查看[issue 9902](http://issue.k8s.io/9902).*


## 细节

### 约束


Serect volume资源要被验证以确保指定的对象引用实际上指向了一个类型为`Secret`的对象。因此，一个secret需要在任何依赖它的pod之前被创建好。

Secret API对象存在于一个namespace中。他们可以被位于相同的namespace的pod引用。

单个的secret的大小限制是1M。这是为了不鼓励创建非常大的secret对象，防止它们耗尽apiserver或者kubelet的内存。然而，创建大量的小体积的secret对象也会让内存耗尽。更加全面的针对内存的限制是一个计划中的特性。

Kubelet只支持从API服务器获取的pod的secret。这包含任何通过kubectl，或者间接通过replication controller创建的pod。这不包含通过kubelet `--manifest-url`标记，`--config`标记或它的REST API创建的Pod（这是不是创建pod的常见做法）。

### 使用Secret的值

在那些挂载了secret volume的容器中，secret的键以文件形式存在，secret的值通过base-64编码并且保存在这些文件中。下面是上面例子中容器中的命令的结果：

```shell
$ ls /etc/foo/
username
password
$ cat /etc/foo/username
value-1
$ cat /etc/foo/password
value-2
```


容器中的程序负责从文件中读取这些secret。目前，如果一个程序想要一个secret保存在环境变量中，那用户需要改动镜像来从这些文件中生成这些环境变量，把这作为运行主程序的预先步骤。未来kubernetes的版本可能会提供更加自动化地从文件中生成环境变量的方式。

### Secret和Pod生命周期的交互

当通过API创建一个pod的时候，引用的secret是否存在是没有被检查的。一旦pod被调度，kubelet会试着去获取secret的值。如果secret因为不存在或者临时的与API服务器连接问题不能被获取到，kubectl会进行周期性的重新尝试。它会报告一个关于pod的事件，解释其未被启动的原因。一旦secret获取成功，kubele会创建并且挂载一个包含该secret的volume。pod中的容器直到volume被挂载之前是不会启动的。

一旦kubelet启动了pod，即使其引用的secret资源被修改了，其挂载的secret volume也不会再改变。要改变使用的secret，必须删除最初的pod，并且创建一个新的pod（很可能`PodSpec`一样）。因此，secret的更新与部署一个新的容器镜像的工作流是一样的。可以使用`kubectl rolling-update`（([man page](/docs/user-guide/kubectl/kubectl_rolling-update)).）。


被secret 引用的时候，它的[`resourceVersion`](https://github.com/kubernetes/kubernetes/tree/{{page.githubbranch}}/docs/devel/api-conventions.md#concurrency-control-and-consistency)字段还没有被指定。因此，如果一个secret的更新和一个pod的启动时间几乎一样，那么哪一个版本的secret会被使用是不确定的。目前还不可能检查使用的是哪个secret对象的resource version。根据计划，pod会报告这个信息，这样replication controller就能重启那些使用老版本的secret的容器。目前，如果这是一个问题，那推荐不要更新现有secret的数据，而是创建新的secret并且选用不同的名字。


## 使用案例

### 使用案例之：有ssh key的Pod

要创建一个pod使用一个把ssh key作为secret的pod，首先要创建一个secret：

```json
{
  "kind": "Secret",
  "apiVersion": "v1",
  "metadata": {
    "name": "ssh-key-secret"
  },
  "data": {
    "id-rsa": "dmFsdWUtMg0KDQo=",
    "id-rsa.pub": "dmFsdWUtMQ0K"
  }
}
```

**提示：** secret数据的序列化的JSON和YAML被编码成base64的字符串。这些字符串中的换行符是不合法的，必须被省略。

现在我们可以创建一个pod，通过sshe key来引用这个secret，并且通过volume来使用它：

```json
{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "secret-test-pod",
    "labels": {
      "name": "secret-test"
    }
  },
  "spec": {
    "volumes": [
      {
        "name": "secret-volume",
        "secret": {
          "secretName": "ssh-key-secret"
        }
      }
    ],
    "containers": [
      {
        "name": "ssh-test-container",
        "image": "mySshImage",
        "volumeMounts": [
          {
            "name": "secret-volume",
            "readOnly": true,
            "mountPath": "/etc/secret-volume"
          }
        ]
      }
    ]
  }
}
```

当容器的命令运行的时候，key的信息将会出现在：

```shell
/etc/secret-volume/id-rsa.pub
/etc/secret-volume/id-rsa
```

然后容器可以使用这个secret的数据来创建ssh连接。

### 使用案例之：在生产/测试环境下使用不同认证信息的pod

这个例子展示了一个pod使用生产环境的认证信息，另一个pod使用测试环境的认证信息：


下面是secret信息：

```json
{
  "apiVersion": "v1",
  "kind": "List",
  "items":
  [{
    "kind": "Secret",
    "apiVersion": "v1",
    "metadata": {
      "name": "prod-db-secret"
    },
    "data": {
      "password": "dmFsdWUtMg0KDQo=",
      "username": "dmFsdWUtMQ0K"
    }
  },
  {
    "kind": "Secret",
    "apiVersion": "v1",
    "metadata": {
      "name": "test-db-secret"
    },
    "data": {
      "password": "dmFsdWUtMg0KDQo=",
      "username": "dmFsdWUtMQ0K"
    }
  }]
}
```

Pod的信息如下:

```json
{
  "apiVersion": "v1",
  "kind": "List",
  "items":
  [{
    "kind": "Pod",
    "apiVersion": "v1",
    "metadata": {
      "name": "prod-db-client-pod",
      "labels": {
        "name": "prod-db-client"
      }
    },
    "spec": {
      "volumes": [
        {
          "name": "secret-volume",
          "secret": {
            "secretName": "prod-db-secret"
          }
        }
      ],
      "containers": [
        {
          "name": "db-client-container",
          "image": "myClientImage",
          "volumeMounts": [
            {
              "name": "secret-volume",
              "readOnly": true,
              "mountPath": "/etc/secret-volume"
            }
          ]
        }
      ]
    }
  },
  {
    "kind": "Pod",
    "apiVersion": "v1",
    "metadata": {
      "name": "test-db-client-pod",
      "labels": {
        "name": "test-db-client"
      }
    },
    "spec": {
      "volumes": [
        {
          "name": "secret-volume",
          "secret": {
            "secretName": "test-db-secret"
          }
        }
      ],
      "containers": [
        {
          "name": "db-client-container",
          "image": "myClientImage",
          "volumeMounts": [
            {
              "name": "secret-volume",
              "readOnly": true,
              "mountPath": "/etc/secret-volume"
            }
          ]
        }
      ]
    }
  }]
}
```

两个容器的文件系统上都有如下的文件存在：

```shell
/etc/secret-volume/username
/etc/secret-volume/password
```

注意上面两个pod的定义只有一个字段不同；这有利于从一个通用的pod配置模板中创建有不同功能的pod。

你可以进一步简化基础的pod定义，通过使用两个service account：例如一个叫做`prod-user`使用`prod-db-secret`，另一个叫做`test-user`使用`test-db-secret`。然后pod的定义可以被简化成如下的样子：

```
{
"kind": "Pod",
"apiVersion": "v1",
"metadata": {
  "name": "prod-db-client-pod",
  "labels": {
    "name": "prod-db-client"
  }
},
"spec": {
  "serviceAccount": "prod-db-client",
  "containers": [
    {
      "name": "db-client-container",
      "image": "myClientImage",
    }
  ]
}
```

### 使用场景之： 对于Pod中一个容器可见的Secret

<a name="use-case-two-containers"></a>

考虑一个需要处理HTTP请求的的程序，它会做些复杂的业务逻辑，然后使用HMAC对一些信息进行签名。因为它的业务逻辑很复杂，服务器中可能有一些不容易注意到的的远程文件读取漏洞，这可能会像攻击者暴露一些敏感的private key。

这可以被切分到两个容器中的两个进程：一个前端容器用户处理用户交互和业务逻辑，但是看不到private key；然后一个signer的容器可以看到private key，只对一些简单的来自前端服务器的签名请求做出响应（如：通过本地的网络）。

使用这种分离式的策略，攻击者不得不使用其他手段来欺骗应用服务器，这可能比让它读取一个文件难的多。

<!-- TODO: explain how to do this while still using automation. -->

## 安全属性

### 保护

因为`secret`的对象可以独立于使用它们的pod进行创建，在pod的创建，查看和编辑的工作流中secret被泄露的风险就更小了。系统也可以对`secret`对象采取额外的预防措施，如防止它们被写到磁盘上可能的地方。

`secret`只有当一个Node上的pod需要它的时候才会送到到该node。它不会被写到磁盘上。它保存在tmpfs上。当使用它的pod删除的时候它也会被删除。

在绝大多数的Kubernetes-project-maintained发行版中，用户和apiserver之间，和从apiserver到kubelet的交流是被SSL/TLS保护着的。secret在这些通道中传输的时候是受到保护的。

node中的secret数据是保存在tmpfs volume中的，因而不会最终保存在node上。

在同一个node上可能有多个pod的secret。然而，只有请求secret的pod包含的容器才能访问它。因此，一个pod不能访问到另外一个pod的secret。

一个pod上可能有很多容器。然而，每一个pod中的容器需要在它的`volumeMounts`请求该secret volume这样才可以在容器中看见它。这可以用来创建有用的 [pod级的安全隔离](#use-case-two-containers).

### 风险

 - 在API Server中，secret数据是在etcd中以明文保存的；因此：
     - 管理员应该限制etcd的访问权限，只让管理人员访问。
     - API服务器中的Secret数据是存放在etcd使用的磁盘上；管理员可能需要对etcd不再使用的时候对它使用过的磁盘进行清除。
 - 应用在读到这些secret之后仍然需要保护secret的值，如防止无意将信息记入日志或者发送到不可信的一方。
 - 用户创建一个使用secret的pod时也可以查看secret的值。即使apiserver的策略不允许该用户读取该secret对象，该用户可以运行一个泄露该secret的pod。
 - 如果多个etcd的副本在运行，那secret在它们之间是共享的。默认情况下，etcd不会对节点之间的通讯使用SSL/TLS进行保护，然而这是可以配置的。
 - 目前还无法控制那些Kubernetes集群的用户可以访问某一个secret。已经有支持这个特性的计划。
 - 目前任何一个node上的root用户都可以通过伪装成kubelet读取所有的secret信息。一个已经计划的特性是只把secret发送给实际上需该secret的node，缩小某个node上滥用root权限带来的影响。
