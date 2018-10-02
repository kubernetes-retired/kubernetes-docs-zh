<!-- ---
title: Documentation Style Guide
content_template: templates/concept
--- -->

---
title: 文档风格指南
content_template: templates/concept
---


{{% capture overview %}}
<!-- This page gives writing style guidelines for the Kubernetes documentation.
These are guidelines, not rules. Use your best judgment, and feel free to
propose changes to this document in a pull request.

For additional information on creating new content for the Kubernetes
docs, follow the instructions on
[using page templates](/docs/home/contribute/page-templates/) and
[creating a documentation pull request](/docs/home/contribute/create-pull-request/). -->

本页提供了 Kubernetes 文档的编写风格指南。这些都是指导原则，而不是规则。发挥你的判断力，并随时通过一个 pull request 对该文档提出修改意见。

关于为 Kubernetes 文档添加新内容的额外信息，请参考 [using page templates](/docs/home/contribute/page-templates/) 和
[creating a documentation pull request](/docs/home/contribute/create-pull-request/)。
{{% /capture %}}

{{% capture body %}}

{{< note >}}
<!-- **Note:** Kubernetes documentation uses [Blackfriday Markdown Renderer](https://github.com/russross/blackfriday) along with a few [Hugo Shortcodes](/docs/home/contribute/includes/) to support glossary entries, tabs, and representing feature state. -->

**注意：** Kubernetes 文档使用 [Blackfriday Markdown Renderer](https://github.com/russross/blackfriday) 以及一些 [Hugo Shortcodes](/docs/home/contribute/includes/) 
来支持术语条目、选项卡和表示特性状态。
{{< /note >}}

<!-- ## Language -->

## 语言

<!-- Kubernetes documentation uses US English. -->

Kubernetes 文档使用美式英语。

<!-- ## Documentation formatting standards -->

## 文档格式化标准

<!-- ### Use camel case for API objects -->

### 使用驼峰写法命名 API

<!-- When you refer to an API object, use the same uppercase and lowercase letters
that are used in the actual object name. Typically, the names of API
objects use
[camel case](https://en.wikipedia.org/wiki/Camel_case). -->

当你要引用 API 时，使用与实际对象名相同的大小写字母。通常，API 的命名使用 [camel case](https://en.wikipedia.org/wiki/Camel_case) 的方式。

<!-- Don't split the API object name into separate words. For example, use
PodTemplateList, not Pod Template List. -->

不要将 API 名拆分成单独的单词。例如，使用 PodTemplateList，而不是 Pod Template List。

<!-- Refer to API objects without saying "object," unless omitting "object"
leads to an awkward construction. -->

在引用 API 时省略“对象”，除非省略“对象”会导致一个笨拙的结构。

<table>
  <tr><th>Do</th><th>Don't</th></tr>
  <tr><td>The Pod has two containers.</td><td>The pod has two containers.</td></tr>
  <tr><td>The Deployment is responsible for ...</td><td>The Deployment object is responsible for ...</td></tr>
  <tr><td>A PodList is a list of Pods.</td><td>A Pod List is a list of pods.</td></tr>
  <tr><td>The two ContainerPorts ...</td><td>The two ContainerPort objects ...</td></tr>
  <tr><td>The two ContainerStateTerminated objects ...</td><td>The two ContainerStateTerminateds ...</td></tr>
</table>

<!-- ### Use angle brackets for placeholders -->

### 使用尖括号表示占位符

<!-- Use angle brackets for placeholders. Tell the reader what a placeholder
represents. -->

使用尖括号表示占位符。告诉读者占位符代表什么。

<!-- 1. Display information about a pod:

       kubectl describe pod <pod-name>

    where `<pod-name>` is the name of one of your pods. -->

1. 查看某个 pod 的相关信息：

       kubectl describe pod <pod-name>

     `<pod-name>` 表示的就是你所要查看的 pod 名称。

<!-- ### Use bold for user interface elements -->

### 使用粗体表示用户界面元素

<table>
  <tr><th>Do</th><th>Don't</th></tr>
  <tr><td>Click <b>Fork</b>.</td><td>Click "Fork".</td></tr>
  <tr><td>Select <b>Other</b>.</td><td>Select 'Other'.</td></tr>
</table>

<!-- ### Use italics to define or introduce new terms -->

### 使用斜体来定义或者引入新的术语

<table>
  <tr><th>Do</th><th>Don't</th></tr>
  <tr><td>A <i>cluster</i> is a set of nodes ...</td><td>A "cluster" is a set of nodes ...</td></tr>
  <tr><td>These components form the <i>control plane.</i></td><td>These components form the <b>control plane.</b></td></tr>
</table>

<!-- ### Use code style for filenames, directories, and paths -->

### 使用代码风格来表示文件名、目录和路径

<table>
  <tr><th>Do</th><th>Don't</th></tr>
  <tr><td>Open the <code>envars.yaml</code> file.</td><td>Open the envars.yaml file.</td></tr>
  <tr><td>Go to the <code>/docs/tutorials</code> directory.</td><td>Go to the /docs/tutorials directory.</td></tr>
  <tr><td>Open the <code>/_data/concepts.yaml</code> file.</td><td>Open the /_data/concepts.yaml file.</td></tr>
</table>

<!-- ### Use the international standard for punctuation inside quotes -->

### 在引号内使用国际标准的标点符号

<table>
  <tr><th>Do</th><th>Don't</th></tr>
  <tr><td>events are recorded with an associated "stage".</td><td>events are recorded with an associated "stage."</td></tr>
  <tr><td>The copy is called a "fork".</td><td>The copy is called a "fork."</td></tr>
</table>

<!-- ## Inline code formatting -->

## 内联代码的格式化

<!-- ### Use code style for inline code and commands -->

### 对于内联代码和命令使用代码样式

<!-- For inline code in an HTML document, use the `<code>` tag. In a Markdown
document, use the backtick (`). -->

对于 HTML 中的内联代码，使用 `<code>` 标签。在 Markdown 的文档中，使用反引号 (`)。

<table>
  <tr><th>Do</th><th>Don't</th></tr>
  <tr><td>The <code>kubectl run</code> command creates a Deployment.</td><td>The "kubectl run" command creates a Deployment.</td></tr>
  <tr><td>For declarative management, use <code>kubectl apply</code>.</td><td>For declarative management, use "kubectl apply".</td></tr>
</table>

<!-- ### Use code style for object field names -->

### 对于对象字段名使用代码样式

<table>
  <tr><th>Do</th><th>Don't</th></tr>
  <tr><td>Set the value of the <code>replicas</code> field in the configuration file.</td><td>Set the value of the "replicas" field in the configuration file.</td></tr>
  <tr><td>The value of the <code>exec</code> field is an ExecAction object.</td><td>The value of the "exec" field is an ExecAction object.</td></tr>
</table>

<!-- ### Use normal style for string and integer field values -->

### 对于字符串和整型字段值使用普通样式

<!-- For field values of type string or integer, use normal style without quotation marks. -->

对于字符串或整型的字段值，使用不带引号的普通样式。

<table>
  <tr><th>Do</th><th>Don't</th></tr>
  <tr><td>Set the value of <code>imagePullPolicy</code> to Always.</td><td>Set the value of <code>imagePullPolicy</code> to "Always".</td></tr>
  <tr><td>Set the value of <code>image</code> to nginx:1.8.</td><td>Set the value of <code>image</code> to <code>nginx:1.8</code>.</td></tr>
  <tr><td>Set the value of the <code>replicas</code> field to 2.</td><td>Set the value of the <code>replicas</code> field to <code>2</code>.</td></tr>
</table>

<!-- ## Code snippet formatting -->

## 代码片段格式化

<!-- ### Don't include the command prompt -->

### 不要包含命令提示符

<table>
  <tr><th>Do</th><th>Don't</th></tr>
  <tr><td>kubectl get pods</td><td>$ kubectl get pods</td></tr>
</table>

<!-- ### Separate commands from output -->

### 从输出中分离命令

<!-- Verify that the pod is running on your chosen node:

    kubectl get pods --output=wide

The output is similar to this:

    NAME     READY     STATUS    RESTARTS   AGE    IP           NODE
    nginx    1/1       Running   0          13s    10.200.0.4   worker0 -->

验证 pod 是否在你选择的节点上运行：

    kubectl get pods --output=wide

输出类似于：

    NAME     READY     STATUS    RESTARTS   AGE    IP           NODE
    nginx    1/1       Running   0          13s    10.200.0.4   worker0

<!-- ### Versioning Kubernetes examples -->

### Kubernetes 版本信息示例

<!-- Code examples and configuration examples that include version information should be consistent with the accompanying text. Identify the Kubernetes version in the **Before you begin** section. -->

包含版本信息的代码示例和配置示例应该与附带的文本保持一致。在 **Before you begin** 部分识别 Kubernetes 版本信息。

<!-- To specify the Kubernetes version for a task or tutorial page, include `min-kubernetes-server-version` in the front matter of the page. -->

为任务或者教程页面指定 Kubernetes 的版本时，需要在页面的页头部分包含 `min-kubernetes-server-version`。

<!-- If the example YAML is in a standalone file, find and review the topics that include it as a reference.
Verify that any topics using the standalone YAML have the appropriate version information defined.
If a stand-alone YAML file is not referenced from any topics, consider deleting it instead of updating it. -->

如果示例 YAML 在单独的文件中，请参考包含该主题的文档。验证使用单独 YAML 的主题是否已经定义了合适的版本信息。如果没有任何主题引用单独的 YAML，考虑是否要删除它而不是更新。

<!-- For example, if you are writing a tutorial that is relevant to Kubernetes version 1.8, the front-matter of your markdown file should look something like: -->

例如：如果你正在编写与 Kubernetes 1.8 相关的教程，则你的 Markdown 文件的页头部分应该如下所示：

```yaml
---
title: <your tutorial title here>
min-kubernetes-server-version: v1.8
---
```

<!-- In code and configuration examples, do not include comments about alternative versions.
Be careful to not include incorrect statements in your examples as comments, such as: -->

在代码和配置示例中，不要包含有关版本替代的注释。小心不要在你的示例中包含错误的语句作为注释，例如：

```yaml
apiVersion: v1 # earlier versions use...
kind: Pod
...
```

<!-- ## Kubernetes.io word list -->

## Kubernetes.io 词汇列表

<!-- A list of Kubernetes-specific terms and words to be used consistently across the site. -->

在整个站点中保持 Kubernetes 特定术语和词汇的一致性。

<table>
  <tr><th>Term</th><th>Usage</th></tr>
  <tr><td>Kubernetes</td><td>Kubernetes should always be capitalized.</td></tr>
  <tr><td>Docker</td><td>Docker should always be capitalized.</td></tr>
  <tr><td>SIG Docs</td><td>SIG Docs rather than SIG-DOCS or other variations.</td></tr>
</table>

<!-- ## Shortcodes -->

## 简码

<!-- Hugo [Shortcodes](https://gohugo.io/content-management/shortcodes) help create different rhetorical appeal levels. Our documentation supports three different shortcodes in this category: **Note:** {{</* note */>}}, **Caution:** {{</* caution */>}}, and **Warning:** {{</* warning */>}}. -->

Hugo [Shortcodes](https://gohugo.io/content-management/shortcodes) 帮助创造不同的修辞层级. 我们的文档支持此类别中的三种不同简码：
**Note:** {{</* note */>}}, **Caution:** {{</* caution */>}}, 和 **Warning:** {{</* warning */>}}.

<!-- 1. Surround the text with an opening and closing shortcode. -->

1. 在文本的开始和结束处都要有简码

<!-- 2. Use the following syntax to apply a style: -->

2. 使用以下的语法来应用样式上：
    
    ```
    {{</* note */>}}
    **Note:** The prefix you use is the same text you use in the tag.
    {{</* /note */>}}
    ```

<!-- The output is: -->

输出如下:

{{< note >}}
**Note:** The prefix you choose is the same text for the tag.
{{< /note >}}

<!-- ### Note -->

### 注意

<!-- Use {{</* note */>}} to highlight a tip or a piece of information that may be helpful to know.

For example: -->

使用 {{</* note */>}} 进行高亮有助于了解提示或信息。

例如：
    
```
{{</* note */>}}
**Note:** You can _still_ use Markdown inside these callouts.
{{</* /note */>}}
```

<!-- The output is: -->

输出如下：

{{< note >}}
**Note:** You can _still_ use Markdown inside these callouts.
{{< /note >}}

<!-- ### Caution -->

### 谨慎

<!-- Use {{</* caution */>}} to call attention to an important piece of information to avoid pitfalls.

For example: -->

使用 {{</* caution */>}} 来提醒重要的信息，以避免陷阱。

例如：

```
{{</* caution */>}}
**Caution:** The callout style only applies to the line directly above the tag.
{{</* /caution */>}}
```

<!-- The output is: -->

输出如下：

{{< caution >}}
**Caution:** The callout style only applies to the line directly above the tag.
{{< /caution >}}

<!-- ### Warning -->

### 警告

<!-- Use {{</* warning */>}} to indicate danger or a piece of information that is crucial to follow.

For example: -->

使用 {{</* warning */>}} 表示危险或至关重要的信息。

例如：

```
{{</* warning */>}}
**Warning:** Beware.
{{</* /warning */>}}
```


<!-- The output is: -->

输出如下：

{{< warning >}}
**Warning:** Beware.
{{< /warning >}}

<!-- ## Common Shortcode Issues -->

## 常见的简码问题

<!-- ### Ordered Lists -->

### 有序列表

<!-- Shortcodes will interrupt numbered lists unless you indent four spaces before the notice and the tag.

For example: -->

除非在提醒和标记之前缩进四个空格，否则简码会中断编号列表。

例如：

    1. Preheat oven to 350˚F

    1. Prepare the batter, and pour into springform pan.
       {{</* note */>}}**Note:** Grease the pan for best results.{{</* /note */>}}

    1. Bake for 20-25 minutes or until set.

<!-- The output is: -->

输出如下：

1. Preheat oven to 350˚F

1. Prepare the batter, and pour into springform pan.
    {{< note >}}**Note:** Grease the pan for best results.{{< /note >}}

1. Bake for 20-25 minutes or until set.

<!-- ## Content best practices -->

## 内容最佳实践

<!-- This section contains suggested best practices for clear, concise, and consistent content. -->

本节包含清晰，简洁和内容一致性的最佳实践。

<!-- ### Use present tense -->

### 使用现在时

<table>
  <tr><th>Do</th><th>Don't</th></tr>
  <tr><td>This command starts a proxy.</td><td>This command will start a proxy.</td></tr>
</table>

<!-- Exception: Use future or past tense if it is required to convey the correct
meaning. -->

例外：有必要的情况下，使用未来时或者过去时来传达正确的含义。

<!-- ### Use active voice -->

### 使用主动语态

<table>
  <tr><th>Do</th><th>Don't</th></tr>
  <tr><td>You can explore the API using a browser.</td><td>The API can be explored using a browser.</td></tr>
  <tr><td>The YAML file specifies the replica count.</td><td>The replica count is specified in the YAML file.</td></tr>
</table>

<!-- Exception: Use passive voice if active voice leads to an awkward construction. -->

例外：如果主动语态会导致笨拙的结构，则使用被动语态。

<!-- ### Use simple and direct language -->

### 使用简单直接的语言

<!-- Use simple and direct language. Avoid using unnecessary phrases, such as saying "please." -->

使用简单直接的语言。避免使用不需要的短语，如 ”请“。

<table>
  <tr><th>Do</th><th>Don't</th></tr>
  <tr><td>To create a ReplicaSet, ...</td><td>In order to create a ReplicaSet, ...</td></tr>
  <tr><td>See the configuration file.</td><td>Please see the configuration file.</td></tr>
  <tr><td>View the Pods.</td><td>With this next command, we'll view the Pods.</td></tr>

</table>

<!-- ### Address the reader as "you" -->

### 将读者称为 ”你“

<table>
  <tr><th>Do</th><th>Don't</th></tr>
  <tr><td>You can create a Deployment by ...</td><td>We'll create a Deployment by ...</td></tr>
    <tr><td>In the preceding output, you can see...</td><td>In the preceding output, we can see ...</td></tr>
</table>

<!-- ### Avoid Latin phrases -->

### 避免拉丁短语

<!-- Prefer English terms over Latin abbreviations. -->

偏向英语术语而不是拉丁缩略词。

<table>
  <tr><th>Do</th><th>Don't</th></tr>
  <tr><td>For example, ...</td><td>e.g., ...</td></tr>
  <tr><td>That is, ...</td><td>i.e., ...</td></tr>
</table>

<!-- Exception: Use "etc." for et cetera. -->

例外：使用 ”etc“ 表示等等。

<!-- ## Patterns to avoid -->

## 要避免的模式

<!-- ### Avoid using "we" -->

### 避免使用 ”我们“

<!-- Using "we" in a sentence can be confusing, because the reader might not know
whether they're part of the "we" you're describing. -->

在语句中使用 ”我们“ 可能会让人疑惑，因为读者可能不清楚他们是否是你所描述的 ”我们“ 的一部分。 

<table>
  <tr><th>Do</th><th>Don't</th></tr>
  <tr><td>Version 1.4 includes ...</td><td>In version 1.4, we have added ...</td></tr>
  <tr><td>Kubernetes provides a new feature for ...</td><td>We provide a new feature ...</td></tr>
  <tr><td>This page teaches you how to use pods.</td><td>In this page, we are going to learn about pods.</td></tr>
</table>

<!-- ### Avoid jargon and idioms -->

### 避免使用行话和俚语

<!-- Some readers speak English as a second language. Avoid jargon and idioms to help them understand better. -->

一部分读者把英语作为第二语言。避免使用行话和俚语能帮助他们更好的理解。

<table>
  <tr><th>Do</th><th>Don't</th></tr>
  <tr><td>Internally, ...</td><td>Under the hood, ...</td></tr>
    <tr><td>Create a new cluster.</td><td>Turn up a new cluster.</td></tr>
</table>

<!-- ### Avoid statements about the future -->

### 避免关于未来的陈述

<!-- Avoid making promises or giving hints about the future. If you need to talk about
an alpha feature, put the text under a heading that identifies it as alpha
information. -->

避免对未来做出承若或者暗示。如果你需要讨论 alpha 功能，将文本放在标识为 alpha 的标题下。

<!-- ### Avoid statements that will soon be out of date -->

### 避免很快就会过时的陈述

<!-- Avoid words like "currently" and "new." A feature that is new today might not be
considered new in a few months. -->

避免使用类似 “当前” 和 “新”。一个今天的新特性，在几个月内可能就不算新了。

<table>
  <tr><th>Do</th><th>Don't</th></tr>
  <tr><td>In version 1.4, ...</td><td>In the current version, ...</td></tr>
    <tr><td>The Federation feature provides ...</td><td>The new Federation feature provides ...</td></tr>
</table>

{{% /capture %}}

{{% capture whatsnext %}}

<!-- * Learn about [writing a new topic](/docs/home/contribute/write-new-topic/).
* Learn about [using page templates](/docs/home/contribute/page-templates/).
* Learn about [staging your changes](/docs/home/contribute/stage-documentation-changes/)
* Learn about [creating a pull request](/docs/home/contribute/create-pull-request/). -->

* 学习关于 [编写一个新的主题](/docs/home/contribute/write-new-topic/).
* 学习关于 [使用页面模板](/docs/home/contribute/page-templates/).
* 学习关于 [暂存你的修改](/docs/home/contribute/stage-documentation-changes/)
* 学习关于 [提交一个 pull request](/docs/home/contribute/create-pull-request/).

{{% /capture %}}
