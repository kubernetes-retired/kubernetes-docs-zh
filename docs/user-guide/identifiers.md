---
---

所有Kubernetes REST API中的对象都可以通过一个Name和一个UID无歧义地辨别。

对于非唯一的，用户提供的属性，Kubernetes提供了[Label](/docs/user-guide/labels)和[Annotation](/docs/user-guide/annotations)。

## Name

Name通常是由客户端提供的。一个给定类型的对象一次只能拥有一个Name(即它们有空间的唯一性)。但是，如果你删除了一个对象，你可以给一个新对象指定该相同的Name。Name用来在Resouce的URL中指代一个对象，例如`/api/v1/pods/some-name`。传统上，Kubernetes资源的名字应该最多不超过253个字符，并且由小写的数字或字母，或者横线(`-`)和点号(`.`)组成，但是有些资源有更为特殊的限制。[Identifiers设计文档](https://github.com/kubernetes/kubernetes/blob/{{page.githubbranch}}/docs/design/identifiers.md)中可以查看到更多精确的命名语法规则。


## UID

UID是由Kubernetes生成的。Kubernetes集群在生命周期中创建的每一个对象都有一个独特的UID（即它们在空间上有暂时的唯一性）。
