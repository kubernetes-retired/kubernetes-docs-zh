<!--
---
title: Configure the aggregation layer
reviewers:
- lavalamp
- cheftako
- chenopis
content_template: templates/task
weight: 10
---
-->
---
title: 配置聚合层
reviewers:
- lavalamp
- cheftako
- chenopis
content_template: templates/task
weight: 10
---

{{% capture overview %}}

<!--
Configuring the [aggregation layer](/docs/concepts/api-extension/apiserver-aggregation/) allows the Kubernetes apiserver to be extended with additional APIs, which are not part of the core Kubernetes APIs. 
-->
配置 [聚合层](/docs/concepts/api-extension/apiserver-aggregation/) 允许使用其他 API 来扩展Kubernetes apiserver，这些 API 不是核心 Kubernetes API 的一部分。

{{% /capture %}}

{{% capture prerequisites %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

<!--
**Note:** There are a few setup requirements for getting the aggregation layer working in your environment to support mutual TLS auth between the proxy and extension apiservers. Kubernetes and the kube-apiserver have multiple CAs, so make sure that the proxy is signed by the aggregation layer CA and not by something else, like the master CA.
-->
**注意：** 在您的环境中需要一直配置来启用聚合层，以支持代理和扩展 apiserver 之前的双向 TLS 认证。Kubernetes 和 kube-apiserver 有多个 CA，因此请确保代理由聚合层 CA 进行签名，而不是由其他东西（如 master CA）。

{{% /capture %}}

{{% capture steps %}}

<!--
## Enable apiserver flags

Enable the aggregation layer via the following kube-apiserver flags. They may have already been taken care of by your provider.

    --requestheader-client-ca-file=<path to aggregator CA cert>
    --requestheader-allowed-names=aggregator
    --requestheader-extra-headers-prefix=X-Remote-Extra-
    --requestheader-group-headers=X-Remote-Group
    --requestheader-username-headers=X-Remote-User
    --proxy-client-cert-file=<path to aggregator proxy cert>
    --proxy-client-key-file=<path to aggregator proxy key>

If you are not running kube-proxy on a host running the API server, then you must make sure that the system is enabled with the following apiserver flag:
-->
## 启用 apiserver 参数

通过以下的 kube-apiserver 参数来启用聚合层。这些有可能已经由您的提供商启用了。

    --requestheader-client-ca-file=<聚合器 CA 证书的路径>
    --requestheader-allowed-names=aggregator
    --requestheader-extra-headers-prefix=X-Remote-Extra-
    --requestheader-group-headers=X-Remote-Group
    --requestheader-username-headers=X-Remote-User
    --proxy-client-cert-file=<聚合器代理证书的路径>
    --proxy-client-key-file=<聚合器代理秘钥的路径>

如果您未在运行 API server 的主机上运行 kube-proxy，则必须确保启用了以下的 apiserver 参数：

    --enable-aggregator-routing=true

{{% /capture %}}

{{% capture whatsnext %}}

<!--
* [Setup an extension api-server](/docs/tasks/access-kubernetes-api/setup-extension-api-server/) to work with the aggregation layer.
* For a high level overview, see [Extending the Kubernetes API with the aggregation layer](/docs/concepts/api-extension/apiserver-aggregation/).
* Learn how to [Extend the Kubernetes API Using Custom Resource Definitions](/docs/tasks/access-kubernetes-api/extend-api-custom-resource-definitions/).
-->
* [配置一个扩展 api-server] (/docs/tasks/access-kubernetes-api/setup-extension-api-server/) 来使用聚合层。
* 有关高级概述，请参阅 [使用聚合层扩展 Kubernetes API](/docs/concepts/api-extension/apiserver-aggregation/)。
* 学习如何 [使用 Custom Resource Definitions 扩展 Kubernetes API](/docs/tasks/access-kubernetes-api/extend-api-custom-resource-definitions/)。

{{% /capture %}}



