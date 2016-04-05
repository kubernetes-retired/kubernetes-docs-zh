---
layout: docwithnav
title: "kubectl annotate"
---

## kubectl annotate

更新某个资源的Annotation

### 摘要

更新一个或多个资源的Annotation。
Annotation是一个键值对，它可以包含比label更多的信息，并且可能是机读数据。
Annotation用来存储那些辅助的，非区分性的信息，特别是那些为外部工具或系统扩展插件使用的数据。
如果--overwrite设为true，将会覆盖现有的Annotation，否则试图修改一个Annotation的值将会抛出错误。
如果设置了--resource-version，那么将会使用指定的这个版本，否则将使用当前版本。

支持的资源包括但不限于（大小写不限）：pods (po)、services (svc)、
replicationcontrollers (rc)、nodes (no)、events (ev)、componentstatuses (cs)、
limitranges (limits)、persistentvolumes (pv)、persistentvolumeclaims (pvc)、
horizontalpodautoscalers (hpa)、resourcequotas (quota)和secrets。

```
{% raw %}
kubectl annotate [--overwrite] (-f FILENAME | TYPE NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--resource-version=version]
{% endraw %}
```

### 示例

```
{% raw %}
# 更新pod “foo”，设置其Annotation description的值为my frontend。
# 如果同一个Annotation被赋值了多次，只保存最后一次设置的值。
$ kubectl annotate pods foo description='my frontend'

# 更新“pod.json”文件中type和name字段指定的pod的Annotation。
$ kubectl annotate -f pod.json description='my frontend'

# 更新pod “foo”，设置其Annotation description的值为my frontend running nginx，已有的值将被覆盖。
$ kubectl annotate --overwrite pods foo description='my frontend running nginx'

# 更新同一namespace下所有的pod。
$ kubectl annotate pods --all description='my frontend running nginx'

# 仅当pod “foo”当前版本为1时，更新其Annotation
$ kubectl annotate pods foo description='my frontend running nginx' --resource-version=1

# 更新pod “foo”，删除其Annotation description。
# 不需要--override选项。
$ kubectl annotate pods foo description-
{% endraw %}
```

### 选项

```
{% raw %}
      --all[=false]: 选择namespace中所有指定类型的资源。
  -f, --filename=[]: 用来指定待升级资源的文件名，目录名或者URL。
      --overwrite[=false]: 如果设置为true，允许覆盖更新Annotation，否则拒绝更新已存在的Annotation。
      --resource-version="": 如果不为空，仅当资源当前版本和指定版本相同时才能更新Annotation。仅当更新单个资源时有效。
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


