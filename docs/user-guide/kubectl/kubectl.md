---
layout: docwithnav
title: "kubectl"
---

## kubectl

使用kubectl来管理Kubernetes集群。

### 摘要

使用kubectl来管理Kubernetes集群。

可以在https://github.com/kubernetes/kubernetes找到更多的信息。

```
{% raw %}
kubectl
{% endraw %}
```

### 选项

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

* [kubectl annotate](kubectl_annotate.md)	 - 更新资源的注解。
* [kubectl api-versions](kubectl_api-versions.md)	 - 以“组/版本”的格式输出服务端支持的API版本。
* [kubectl apply](kubectl_apply.md)	 - 通过文件名或控制台输入，对资源进行配置。
* [kubectl attach](kubectl_attach.md)	 - 连接到一个正在运行的容器。
* [kubectl autoscale](kubectl_autoscale.md)	 - 对replication controller进行自动伸缩。
* [kubectl cluster-info](kubectl_cluster-info.md)	 - 输出集群信息。
* [kubectl config](kubectl_config.md)	 - 修改kubeconfig配置文件。
* [kubectl create](kubectl_create.md)	 - 通过文件名或控制台输入，创建资源。
* [kubectl delete](kubectl_delete.md)	 - 通过文件名、控制台输入、资源名或者label selector删除资源。
* [kubectl describe](kubectl_describe.md)	 - 输出指定的一个/多个资源的详细信息。
* [kubectl edit](kubectl_edit.md)	 - 编辑服务端的资源。
* [kubectl exec](kubectl_exec.md)	 - 在容器内部执行命令。
* [kubectl expose](kubectl_expose.md)	 - 输入replication controller、service或者pod，并将其暴露为新的kubernetes service。
* [kubectl get](kubectl_get.md)	 - 输出一个/多个资源。
* [kubectl label](kubectl_label.md)	 - 更新资源的label。
* [kubectl logs](kubectl_logs.md)	 - 输出pod中一个容器的日志。
* [kubectl namespace](kubectl_namespace.md)	 -（已停用）设置或查看当前使用的namespace。
* [kubectl patch](kubectl_patch.md)	 - 通过控制台输入更新资源中的字段。
* [kubectl port-forward](kubectl_port-forward.md)	 - 设置本地端口到Pod的转发。
* [kubectl proxy](kubectl_proxy.md)	 - 为Kubernetes API server启动代理服务器。
* [kubectl replace](kubectl_replace.md)	 - 通过文件名或控制台输入替换资源。
* [kubectl rolling-update](kubectl_rolling-update.md)	 - 对指定的replication controller执行滚动升级。
* [kubectl run](kubectl_run.md)	 - 在集群中使用指定镜像启动容器。
* [kubectl scale](kubectl_scale.md)	 - 为replication controller设置新的副本数。
* [kubectl stop](kubectl_stop.md)	 - （已停用）通过资源名或控制台输入安全的删除资源。
* [kubectl version](kubectl_version.md)	 - 输出服务端和客户端的版本信息。
