---
title: Installing kubeadm
content_template: templates/task
weight: 20
---

{{% capture overview %}}

<!--
<img src="https://raw.githubusercontent.com/cncf/artwork/master/kubernetes/certified-kubernetes/versionless/color/certified-kubernetes-color.png" align="right" width="150px">This page shows how to install the `kubeadm` toolbox.
For information how to create a cluster with kubeadm once you have performed this installation process,
see the [Using kubeadm to Create a Cluster](/docs/setup/independent/create-cluster-kubeadm/) page.
-->
<img src="https://raw.githubusercontent.com/cncf/artwork/master/kubernetes/certified-kubernetes/versionless/color/certified-kubernetes-color.png" align="right" width="150px">本文会告诉您如何安装 `kubeadm` 工具。完成本文提到的安装步骤后，您可以阅读 [使用 kubeadm 来创建集群](/docs/setup/independent/create-cluster-kubeadm/) 了解如何使用 kubeadm 来创建集群。


{{% /capture %}}

{{% capture prerequisites %}}
<!--
* One or more machines running one of:
  - Ubuntu 16.04+
  - Debian 9
  - CentOS 7
  - RHEL 7
  - Fedora 25/26 (best-effort)
  - HypriotOS v1.0.1+
  - Container Linux (tested with 1576.4.0)
* 2 GB or more of RAM per machine (any less will leave little room for your apps)
* 2 CPUs or more 
* Full network connectivity between all machines in the cluster (public or private network is fine)
* Unique hostname, MAC address, and product_uuid for every node. See [here](#verify-the-mac-address-and-product-uuid-are-unique-for-every-node) for more details.
* Certain ports are open on your machines. See [here](#check-required-ports) for more details.
* Swap disabled. You **MUST** disable swap in order for the kubelet to work properly. 
-->
* 一台或多台运行着下列系统的机器:
  - Ubuntu 16.04+
  - Debian 9
  - CentOS 7
  - RHEL 7
  - Fedora 25/26 (最佳实践)
  - HypriotOS v1.0.1+
  - Container Linux (1576.4.0 版本可用)
* 每台机器 2 GB 或更多的 RAM (如果少于这个数字将会影响您应用的运行内存)
* 2 CPU 核心或更多 
* 集群中的所有机器的网络可以连通(公网和内网都可以)
* 节点之中不可以有重复的主机名，MAC 地址，产品序列号。更多详细信息请参见[这里](#verify-the-mac-address-and-product-uuid-are-unique-for-every-node) 。
* 开启主机上的一些特定端口. 更多详细信息请参见[这里](#check-required-ports)。
* 关闭 Swap。为了保证 kubelet 正确运行，您 **必须** 关闭 swap。 

{{% /capture %}}

{{% capture steps %}}

<!--
## Verify the MAC address and product_uuid are unique for every node
-->
## 保证每个节点上 MAC 地址和 product_uuid 是唯一的

<!--
* You can get the MAC address of the network interfaces using the command `ip link` or `ifconfig -a`
* The product_uuid can be checked by using the command `sudo cat /sys/class/dmi/id/product_uuid`
-->
* 您可以使用下列命令获取网络接口的 MAC 地址 `ip link` 或是 `ifconfig -a`
* 下列命令可以用来获取 product_uuid `sudo cat /sys/class/dmi/id/product_uuid`

<!--
It is very likely that hardware devices will have unique addresses, although some virtual machines may have
identical values. Kubernetes uses these values to uniquely identify the nodes in the cluster.
If these values are not unique to each node, the installation process
may [fail](https://github.com/kubernetes/kubeadm/issues/31).
-->
一般来讲，硬件设备会拥有独一无二的地址，但是有些虚拟机可能会雷同。Kubernetes 使用这些值来唯一确定集群中的节点。如果这些值在集群中不为意，安装可能会[失败](https://github.com/kubernetes/kubeadm/issues/31)。

<!--
## Check network adapters
-->
## 检查网络适配器

<!--
If you have more than one network adapter, and your Kubernetes components are not reachable on the default
route, we recommend you add IP route(s) so Kubernetes cluster addresses go via the appropriate adapter.
-->
如果您有一个以上的网络适配器，同时您的Kubernetes组件在默认情况下不可达，我们建议您预先配置好 IP 路径，这样 Kubernetes 集群就可以通过适当的适配器。

<!--
## Check required ports
-->
## 检查所需端口

<!--
### Master node(s)
-->
### Master 节点

| Protocol | Direction | Port Range | Purpose                 | Used By                   |
|----------|-----------|------------|-------------------------|---------------------------|
| TCP      | Inbound   | 6443*      | Kubernetes API server   | All                       |
| TCP      | Inbound   | 2379-2380  | etcd server client API  | kube-apiserver, etcd      |
| TCP      | Inbound   | 10250      | Kubelet API             | Self, Control plane       |
| TCP      | Inbound   | 10251      | kube-scheduler          | Self                      |
| TCP      | Inbound   | 10252      | kube-controller-manager | Self                      |

<!--
### Worker node(s)
-->
### Worker 节点

| Protocol | Direction | Port Range  | Purpose               | Used By                 |
|----------|-----------|-------------|-----------------------|-------------------------|
| TCP      | Inbound   | 10250       | Kubelet API           | Self, Control plane     |
| TCP      | Inbound   | 30000-32767 | NodePort Services**   | All                     |

<!--
** Default port range for [NodePort Services](/docs/concepts/services-networking/service/).
-->
** [NodePort 服务](/docs/concepts/services-networking/service/) 的默认端口范围。

<!--
Any port numbers marked with * are overridable, so you will need to ensure any
custom ports you provide are also open.
-->
任意使用 * 标记的端口号都可以被覆盖，所以您需要保证自定端口是开启的。

<!--
Although etcd ports are included in master nodes, you can also host your own
etcd cluster externally or on custom ports.
-->
虽然 master 节点已经包含了 etcd 的端口，您也可以使用自定义的外部 etcd 集群，并且指定自定义端口。

<!--
The pod network plugin you use (see below) may also require certain ports to be
open. Since this differs with each pod network plugin, please see the
documentation for the plugins about what port(s) those need.
-->
您使用的 pod network plugin (见下) 也可能需要某些特定端口开启。由于各个 pod network plugin 都有所不同，请参阅他们各自文档中对端口的要求。

<!--
## Installing Docker
-->
## 安装 Docker

<!--
On each of your machines, install Docker.
Version 17.03 is recommended, but 1.11, 1.12 and 1.13 are known to work as well.
Versions 17.06+ _might work_, but have not yet been tested and verified by the Kubernetes node team.
Keep track of the latest verified Docker version in the Kubernetes release notes.
-->
在每一台机器上安装 Docker。
推荐使用 17.03 版本，不过 1.11, 1.12 和 1.13 也同样支持。.
版本 17.06 以上 _可能被支持_, 但是还没有被我们的 Kubernetes node 团队验证过。
请关注 Kubernetes 的更新文档以查看最新支持的 Docker 版本。

<!--
Please proceed with executing the following commands based on your OS as root. You may become the root user by executing `sudo -i` after SSH-ing to each host.
-->
请按照您的系统使用相应的命令来转换为 root 用户。您可以在 SSH 到主机之后使用 `sudo -i` 命令从而转为 root 用户。

<!--
If you already have the required versions of the Docker installed, you can move on to next section.
If not, you can use the following commands to install Docker on your system:
-->
如果您已经安装了相应版本，您可以跳过直接看到下一节。
否则，您可以使用下列命令安装 Docker。

<!--
{{< tabs name="docker_install" >}}
{{% tab name="Ubuntu, Debian or HypriotOS" %}}
Install Docker from Ubuntu's repositories:
-->
{{< tabs name="docker_install" >}}
{{% tab name="Ubuntu, Debian or HypriotOS" %}}
从 Ubuntu 仓库安装 Docker：

```bash
apt-get update
apt-get install -y docker.io
```

<!--
or install Docker CE 17.03 from Docker's repositories for Ubuntu or Debian:
-->
或者从 Docker 仓库安装 Ubuntu 或 Debian 系统对应的 Docker CE 17.03：

```bash
apt-get update
apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") $(lsb_release -cs) stable"
apt-get update && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.03 | head -1 | awk '{print $3}')
```
<!--
{{% /tab %}}
{{% tab name="CentOS, RHEL or Fedora" %}}
Install Docker using your operating system's bundled package:
-->
{{% /tab %}}
{{% tab name="CentOS, RHEL or Fedora" %}}
使用操作系统内置安装包安装 Docker：

```bash
yum install -y docker
systemctl enable docker && systemctl start docker
```
<!--
{{% /tab %}}
{{% tab name="Container Linux" %}}
Enable and start Docker:
-->
{{% /tab %}}
{{% tab name="Container Linux" %}}
启用并启动 Docker：

```bash
systemctl enable docker && systemctl start docker
```
{{% /tab %}}
{{< /tabs >}}


<!--
Refer to the [official Docker installation guides](https://docs.docker.com/engine/installation/)
for more information.
-->
请参阅 [Docker 官方安装文档](https://docs.docker.com/engine/installation/) 获取更多信息。

<!--
## Installing kubeadm, kubelet and kubectl
-->
## 安装 kubeadm, kubelet 和 kubectl

<!--
You will install these packages on all of your machines:

* `kubeadm`: the command to bootstrap the cluster.

* `kubelet`: the component that runs on all of the machines in your cluster
    and does things like starting pods and containers.

* `kubectl`: the command line util to talk to your cluster.
-->
您需要在每台机器上都安装以下的软件包：

* `kubeadm`: 用来初始化集群的指令。

* `kubelet`: 在集群中的每个节点上用来启动 pod 和 container 等。

* `kubectl`: 用来与集群通信的命令行工具。

<!--
kubeadm **will not** install or manage `kubelet` or `kubectl` for you, so you will 
need to ensure they match the version of the Kubernetes control panel you want 
kubeadm to install for you. If you do not, there is a risk of a version skew occurring that
can lead to unexpected, buggy behaviour. However, _one_ minor version skew between the 
kubelet and the control plane is supported, but the kubelet version may never exceed the API
server version. For example, kubelets running 1.7.0 should be fully compatible with a 1.8.0 API server,
but not vice versa.
-->
kubeadm **不能** 帮您安装或管理 `kubelet` 或 `kubectl` ，所以您得保证他们满足通过 kubeadm 安装的 Kubernetes 控制层对版本的要求。如果版本没有满足要求，就有可能导致一些难以想到的错误或问题。然而 _一个_ 轻微的 控制层与 kubelet间的版本不一致无伤大雅，不过请记住 kubelet 的版本不可以超过 API server 的版本。例如 1.8.0 的 API server 可以适配 1.7.0 的 kubelet，反之就不行了。

<!--
For more information on version skews, please read our 
[version skew policy](/docs/setup/independent/create-cluster-kubeadm/#version-skew-policy).
-->
更多关于版本冲突的信息，请参阅 [版本冲突政策](/docs/setup/independent/create-cluster-kubeadm/#version-skew-policy)。

<!--
{{< tabs name="k8s_install" >}}
{{% tab name="Ubuntu, Debian or HypriotOS" %}}
-->
{{< tabs name="k8s_安装" >}}
{{% tab name="Ubuntu, Debian 或 HypriotOS" %}}
```bash
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
```
{{% /tab %}}
{{% tab name="CentOS, RHEL or Fedora" %}}
```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
setenforce 0
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
```

<!--
  **Note:**
-->
  **注意：**

<!--
  - Disabling SELinux by running `setenforce 0` is required to allow containers to access the host filesystem, which is required by pod networks for example.
    You have to do this until SELinux support is improved in the kubelet.
  - Some users on RHEL/CentOS 7 have reported issues with traffic being routed incorrectly due to iptables being bypassed. You should ensure
    `net.bridge.bridge-nf-call-iptables` is set to 1 in your `sysctl` config, e.g.
-->
  - 为了使 container 可以访问宿主机的文件系统，请使用 `setenforce 0` 来禁用SELinux。这是 pod network 的要求。
    您必须这么做，直到 kubelet 支持 SELinux 为止。
  - 一些 RHEL/CentOS 7 曾经遇到过受到 iptable 被绕过的影响，网络请求被错误的路由。您得保证
    在您的 `sysctl` 配置中 `net.bridge.bridge-nf-call-iptables` 被设为1。
  
    ```bash
    cat <<EOF >  /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF
    sysctl --system
    ```
{{% /tab %}}
{{% tab name="Container Linux" %}}
<!--
Install CNI plugins (required for most pod network):
-->
安装 CNI 插件（大多数 pod network 都需要）

```bash
CNI_VERSION="v0.6.0"
mkdir -p /opt/cni/bin
curl -L "https://github.com/containernetworking/plugins/releases/download/${CNI_VERSION}/cni-plugins-amd64-${CNI_VERSION}.tgz" | tar -C /opt/cni/bin -xz
```

<!--
Install `kubeadm`, `kubelet`, `kubectl` and add a `kubelet` systemd service:
-->
安装 `kubeadm`, `kubelet`, `kubectl` 并添加 `kubelet` 为 systemd 服务：

```bash
RELEASE="$(curl -sSL https://dl.k8s.io/release/stable.txt)"

mkdir -p /opt/bin
cd /opt/bin
curl -L --remote-name-all https://storage.googleapis.com/kubernetes-release/release/${RELEASE}/bin/linux/amd64/{kubeadm,kubelet,kubectl}
chmod +x {kubeadm,kubelet,kubectl}

curl -sSL "https://raw.githubusercontent.com/kubernetes/kubernetes/${RELEASE}/build/debs/kubelet.service" | sed "s:/usr/bin:/opt/bin:g" > /etc/systemd/system/kubelet.service
mkdir -p /etc/systemd/system/kubelet.service.d
curl -sSL "https://raw.githubusercontent.com/kubernetes/kubernetes/${RELEASE}/build/debs/10-kubeadm.conf" | sed "s:/usr/bin:/opt/bin:g" > /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

<!--
Enable and start `kubelet`:
-->
启用并启动 `kubelet`：

```bash
systemctl enable kubelet && systemctl start kubelet
```
{{% /tab %}}
{{< /tabs >}}


<!--
The kubelet is now restarting every few seconds, as it waits in a crashloop for
kubeadm to tell it what to do.
-->
kubelet 现在每隔几秒就会重启，因为它陷入了一个等待 kubeadm 指令的死循环。

<!--
## Configure cgroup driver used by kubelet on Master Node
-->
## 在 Master 节点上配置 kubelet 所需的 cgroup driver

<!--
When using Docker, kubeadm will automatically detect the cgroup driver for the kubelet
and set it in the `/var/lib/kubelet/kubeadm-flags.env` file during runtime.
-->
使用 Docker 时，kubeadm 会自动为其检测 cgroup driver 并实时对 `/var/lib/kubelet/kubeadm-flags.env` 文件进行配置。

<!--
If you are using a different CRI, you have to modify the file
`/etc/default/kubelet` with your `cgroup-driver` value, like so:
-->
如果您使用了不同的 CRI， 您得把 `/etc/default/kubelet` 文件中的 `cgroup-driver` 位置改为对应的值，像这样：

```bash
KUBELET_KUBEADM_EXTRA_ARGS=--cgroup-driver=<value>
```

<!--
This file will be used by `kubeadm init` and `kubeadm join` to source extra
user defined arguments for the kubelet.
-->
这个文件将会被 `kubeadm init` 和 `kubeadm join` 用于为 kubelet source 额外的用户参数。

<!--
Please mind, that you **only** have to do that if the cgroup driver of your CRI
is not , because that is the default value in the kubelet already.
-->
请注意，您**只**需要在您的 cgroup driver 不是 `cgroupfs` 时这么做，因为那是 kubelet 的默认值。

<!--
Restarting the kubelet is required:
-->
需要重启 kubelet：

```bash
systemctl daemon-reload
systemctl restart kubelet
```

<!--
## Troubleshooting
-->
## 查错

<!--
If you are running into difficulties with kubeadm, please consult our [troubleshooting docs](/docs/setup/independent/troubleshooting-kubeadm/).
-->
如果您在使用 kubeadm 时候遇到问题，请查看我们的[疑难解答文档](/docs/setup/independent/troubleshooting-kubeadm/)。

{{% capture whatsnext %}}

<!--
* [Using kubeadm to Create a Cluster](/docs/setup/independent/create-cluster-kubeadm/)
-->
* [使用 kubeadm 来创建集群](/docs/setup/independent/create-cluster-kubeadm/)

{{% /capture %}}





