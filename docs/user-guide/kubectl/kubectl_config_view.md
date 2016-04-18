---
layout: docwithnav
title: "kubectl config view"
---
## kubectl config view

显示合并后的kubeconfig设置，或者一个指定的kubeconfig配置文件。

### 摘要

显示合并后的kubeconfig设置，或者一个指定的kubeconfig配置文件。
用户可使用--output=template --template=TEMPLATE来选择输出指定的值。

```
{% raw %}
kubectl config view
{% endraw %}
```

### 示例

```
{% raw %}
# 显示合并后的kubeconfig设置
$ kubectl config view

# 获取e2e用户的密码
$ kubectl config view -o template --template='{{range .users}}{{ if eq .name "e2e" }}{{ index .user.password }}{{end}}{{end}}'
{% endraw %}
```

### 选项

```
{% raw %}
      --flatten[=false]: 将读取的kubeconfig配置文件扁平输出为自包含的结构（对创建可迁移的kubeconfig配置文件有帮助）
      --merge=true: 按照继承关系合并所有的kubeconfig配置文件。
      --minify[=false]: 如果为true，不显示目前环境未使用到的任何信息。
      --no-headers[=false]: 当使用默认输出格式时不打印标题栏。
  -o, --output="": 输出格式，只能使用json|yaml|wide|name|go-template=...|go-template-file=...|jsonpath=...|jsonpath-file=...中的一种。参见golang模板[http://golang.org/pkg/text/template/#pkg-overview]和jsonpath模板[http://releases.k8s.io/release-1.1/docs/user-guide/jsonpath.md]。
      --output-version="": 输出资源使用的API版本（默认使用api-version）。
      --raw[=false]: 显示未经格式化的字节信息。
  -a, --show-all[=false]: 打印输出时，显示所有的资源（默认隐藏状态为terminated的pod）。
      --sort-by="": 如果不为空，对输出的多个结果根据指定字段进行排序。该字段使用jsonpath表达式（如“ObjectMeta.Name”）描述，并且该字段只能为字符串或者整数类型。
      --template="": 当指定了-o=go-template或-o=go-template-file时使用的模板字符串或者模板文件。模板的格式为golang模板[http://golang.org/pkg/text/template/#pkg-overview]。
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

* [kubectl config](kubectl_config.html)	 - 修改kubeconfig配置文件。
