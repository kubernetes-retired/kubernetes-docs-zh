---
layout: docwithnav
title: "kubectl config"
---

## kubectl config

修改kubeconfig配置文件。

### 摘要

使用config的子命令修改kubeconfig配置文件，如“kubectl config set current-context my-context”。

配置文件的读取顺序遵循如下规则：

    1. 如果指定了--kubeconfig选项，那么只有指定的文件被加载。此选项只能被设置一次，并且不会合并其他文件。
    2. 如果设置了$KUBECONFIG环境变量，将同时使用此环境变量指定的所有文件列表（使用操作系统默认的文件分隔规则），所有文件将被合并。当修改一个值时，将修改设置了该值的文件。当创建一个值时，将在列表的首个文件创建该值。若列表中所有的文件都不存在，将创建列表中的最后一个文件。
    3. 如果前两项都没有设置,将使用 ${HOME}/.kube/config，并且不会合并其他文件。


```
{% raw %}
kubectl config SUBCOMMAND
{% endraw %}
```

### 选项

```
{% raw %}
      --kubeconfig="": 使用指定的kubeconfig配置文件。
{% endraw %}
```

### 继承自父命令的选项

```
{% raw %}
      --alsologtostderr[=false]: 同时输出日志到标准错误控制台和文件。
      --api-version="": 和服务端交互使用的API版本。
      --certificate-authority="": 用以进行证书权威性认证的.cert文件路径。
      --client-certificate="": TLS使用的客户端证书路径。
      --client-key="": TLS使用的客户端密钥路径。
      --cluster="": 指定使用的kubeconfig配置文件中的集群名。
      --context="": 指定使用的kubeconfig配置文件中的环境名。
      --insecure-skip-tls-verify[=false]: 如果为true，将不会检查服务器凭证的有效性，这会导致你的HTTPS链接变得不安全。
      --kubeconfig="": 命令行请求使用的配置文件路径。
      --log-backtrace-at=:0: 当日志长度超过定义的行数时，忽略堆栈信息。
      --log-dir="": 如果不为空，将日志文件写入此目录。
      --log-flush-frequency=5s: 刷新日志的最大时间间隔。
      --logtostderr[=true]: 输出日志到标准错误控制台，不输出到文件。
      --match-server-version[=false]: 要求服务端和客户端版本匹配。
      --namespace="": 如果不为空，命令将使用此namespace。
      --password="": API Server进行简单认证使用的密码。
  -s, --server="": Kubernetes API Server的地址和端口号。
      --stderrthreshold=2: 高于此级别的日志将被输出到错误控制台。
      --token="": 认证到API Server使用的令牌。
      --user="": 指定使用的kubeconfig配置文件中的用户名。
      --username="": API Server进行简单认证使用的用户名。
      --v=0: 指定输出日志的级别。
      --vmodule=: 指定输出日志的模块，格式如下：pattern=N，使用逗号分隔。
{% endraw %}
```

### 参见

* [kubectl](kubectl.html)	 - 使用kubectl来管理Kubernetes集群。
* [kubectl config set](kubectl_config_set.html)	 - 在kubeconfig配置文件中设置一个单独的值。
* [kubectl config set-cluster](kubectl_config_set-cluster.html)	 - 在kubeconfig配置文件中设置一个集群项。
* [kubectl config set-context](kubectl_config_set-context.html)	 - 在kubeconfig配置文件中设置一个环境项。
* [kubectl config set-credentials](kubectl_config_set-credentials.html)	 - 在kubeconfig配置文件中设置一个用户项。
* [kubectl config unset](kubectl_config_unset.html)	 - 在kubeconfig配置文件中清除一个单独的值。
* [kubectl config use-context](kubectl_config_use-context.html)	 - 使用kubeconfig中的一个环境项作为当前配置。
* [kubectl config view](kubectl_config_view.html)	 - 显示合并后的kubeconfig设置，或者一个指定的kubeconfig配置文件。
