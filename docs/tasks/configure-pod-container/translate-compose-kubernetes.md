---
reviewers:
- cdrage
title: 转化 Docker Compose 文件为 Kubernetes 资源
content_template: templates/task
weight: 170
---

<!--

---
reviewers:
- cdrage
title: Translate a Docker Compose File to Kubernetes Resources
content_template: templates/task
weight: 170
---

-->

{{% capture overview %}}

<!--
What's Kompose? It's a conversion tool for all things compose (namely Docker Compose) to container orchestrators (Kubernetes or OpenShift).
-->
什么是 Kompose？ 它是一个迁移工具，用于将 compose相关资源 （即 Docker Compose ）转换为其他容器编排系统资源（比如 Kubernetes 或者 Openshift ）。

<!--
More information can be found on the Kompose website at [http://kompose.io](http://kompose.io).
-->
在 [http://kompose.io](http://kompose.io) 可以查看更多的信息。

{{% /capture %}}

{{< toc >}}

{{% capture prerequisites %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

{{% /capture %}}

{{% capture steps %}}

<!--
## Install Kompose
-->
## 安装 Kompose

<!--
We have multiple ways to install Kompose. Our preferred method is downloading the binary from the latest GitHub release.
-->
在众多安装 Kompose 的方法中，推荐的安装方法是从 GitHub 中下载最新的二进制版本来安装。

<!--
## GitHub release
-->
## GitHub 版本

<!--
Kompose is released via GitHub on a three-week cycle, you can see all current releases on the [GitHub release page](https://github.com/kubernetes/kompose/releases).
-->
Kompose 在 GitHub 上每三周发版一次，可以从 [GitHub release page](https://github.com/kubernetes/kompose/releases) 中查看所有的发行版。

```sh
# Linux 
curl -L https://github.com/kubernetes/kompose/releases/download/v1.1.0/kompose-linux-amd64 -o kompose

# macOS
curl -L https://github.com/kubernetes/kompose/releases/download/v1.1.0/kompose-darwin-amd64 -o kompose

# Windows
curl -L https://github.com/kubernetes/kompose/releases/download/v1.1.0/kompose-windows-amd64.exe -o kompose.exe

chmod +x kompose
sudo mv ./kompose /usr/local/bin/kompose
```

<!--
Alternatively, you can download the [tarball](https://github.com/kubernetes/kompose/releases).
-->
或者，可以从 [tarball](https://github.com/kubernetes/kompose/releases) 下载。

## Go

<!--
Installing using `go get` pulls from the master branch with the latest development changes.
-->
使用 `go get` 从 master 分支中拉取最新的代码来安装。

```sh
go get -u github.com/kubernetes/kompose
```

## CentOS

<!--
Kompose is in [EPEL](https://fedoraproject.org/wiki/EPEL) CentOS repository.
If you don't have [EPEL](https://fedoraproject.org/wiki/EPEL) repository already installed and enabled you can do it by running  `sudo yum install epel-release`
-->
Kompose 安装包在 [EPEL](https://fedoraproject.org/wiki/EPEL)CentOS 仓库。
如果没有 [EPEL](https://fedoraproject.org/wiki/EPEL) 仓库，执行 `sudo yum install epel-release` 安装。

<!--
If you have [EPEL](https://fedoraproject.org/wiki/EPEL) enabled in your system, you can install Kompose like any other package.
-->
如果有 [EPEL](https://fedoraproject.org/wiki/EPEL)，可以像安装其他安装包一样来安装 Kompose。

```bash
sudo yum -y install kompose
```

## Fedora

<!--
Kompose is in Fedora 24, 25 and 26 repositories. You can install it just like any other package.
-->
Kompose 在 Fedora 24, 25 和 26 仓库，可以向安装其他安装包一样来安装 Kompose。

```bash
sudo dnf -y install kompose
```

## macOS

<!--
On macOS you can install latest release via [Homebrew](https://brew.sh):
-->
在 macOS，可以通过 [Homebrew](https://brew.sh) 安装最新发行版：

```bash
brew install kompose

```

<!--
## Use Kompose
-->
## 使用 Kompose

<!--
In just a few steps, we'll take you from Docker Compose to Kubernetes. All
you need is an existing `docker-compose.yml` file.
-->
仅仅需要一个已经存在的 `docker-compose.yml` 文件，几个步骤内，就可以从 Docker Compose 迁移到 Kubernetes。

<!--
1.  Go to the directory containing your `docker-compose.yml` file. If you don't
    have one, test using this one.
-->
1.  进入包含 `docker-compose.yml` 文件的目录。如果没有，使用下面的 `docker-compose.yml` 文件来测试。

      ```yaml
      version: "2"

      services:

        redis-master:
          image: k8s.gcr.io/redis:e2e 
          ports:
            - "6379"

        redis-slave:
          image: gcr.io/google_samples/gb-redisslave:v1
          ports:
            - "6379"
          environment:
            - GET_HOSTS_FROM=dns

        frontend:
          image: gcr.io/google-samples/gb-frontend:v4
          ports:
            - "80:80"
          environment:
            - GET_HOSTS_FROM=dns
          labels:
            kompose.service.type: LoadBalancer
      ```

<!--
2.  Run the `kompose up` command to deploy to Kubernetes directly, or skip to
    the next step instead to generate a file to use with `kubectl`.
-->
2.  运行 `kompose up` 命令直接安装到 Kubernetes，或者跳到下一步骤，通过 `kubectl` 命令来安装。

      ```bash
      $ kompose up
      We are going to create Kubernetes Deployments, Services and PersistentVolumeClaims for your Dockerized application. 
      If you need different kind of resources, use the 'kompose convert' and 'kubectl create -f' commands instead. 
      

      INFO Successfully created Service: redis          
      INFO Successfully created Service: web            
      INFO Successfully created Deployment: redis       
      INFO Successfully created Deployment: web         

      Your application has been deployed to Kubernetes. You can run 'kubectl get deployment,svc,pods,pvc' for details.
      ```

<!--
3.  To convert the `docker-compose.yml` file to files that you can use with
    `kubectl`, run `kompose convert` and then `kubectl create -f <output file>`.
-->
3.  通过 `kompose convert` 命令将 `docker-compose.yml` 文件转换成 `kubectl` 可识别的文件，然后 `kubectl create -f <output file>`。

      ```bash
      $ kompose convert                           
      INFO Kubernetes file "frontend-service.yaml" created         
      INFO Kubernetes file "redis-master-service.yaml" created     
      INFO Kubernetes file "redis-slave-service.yaml" created      
      INFO Kubernetes file "frontend-deployment.yaml" created      
      INFO Kubernetes file "redis-master-deployment.yaml" created  
      INFO Kubernetes file "redis-slave-deployment.yaml" created   
      ```

      ```bash
      $ kubectl create -f frontend-service.yaml,redis-master-service.yaml,redis-slave-service.yaml,frontend-deployment.yaml,redis-master-deployment.yaml,redis-slave-deployment.yaml
      service "frontend" created
      service "redis-master" created
      service "redis-slave" created
      deployment "frontend" created
      deployment "redis-master" created
      deployment "redis-slave" created

      ```
<!--      
      Your deployments are running in Kubernetes.
-->
     您的deployments正运行在Kubernetes上。

<!--
4.  Access your application.
-->
4.  访问应用程序

<!--
      If you're already using `minikube` for your development process:
-->
      如果您已经使用 `minikube` 部署了 Kubernetes：

      ```bash
      $ minikube service frontend
      ```

<!--
      Otherwise, let's look up what IP your service is using!
-->
      否则，查看您的服务使用的什么IP！

      ```sh
      $ kubectl describe svc frontend
      Name:                   frontend
      Namespace:              default
      Labels:                 service=frontend
      Selector:               service=frontend
      Type:                   LoadBalancer
      IP:                     10.0.0.183
      LoadBalancer Ingress:   123.45.67.89
      Port:                   80      80/TCP
      NodePort:               80      31144/TCP
      Endpoints:              172.17.0.4:80
      Session Affinity:       None
      No events.

      ```

<!--
      If you're using a cloud provider, your IP will be listed next to `LoadBalancer Ingress`.
-->
      如果您使用了云提供商，IP会列在 `LoadBalancer Ingress` 后面。

      ```sh
      $ curl http://123.45.67.89
      ```

{{% /capture %}}

{{% capture discussion %}}

<!--
## User Guide
-->
## 用户指南

- CLI
  - [`kompose convert`](#kompose-convert)
  - [`kompose up`](#kompose-up)
  - [`kompose down`](#kompose-down)
<!--
- Documentation
  - [Build and Push Docker Images](#build-and-push-docker-images)
  - [Alternative Conversions](#alternative-conversions)
  - [Labels](#labels)
  - [Restart](#restart)
  - [Docker Compose Versions](#docker-compose-versions)
-->
- 文档
  - [构建并推送 docker 镜像](#构建并推送-docker-镜像)
  - [替代转换](#替代转换)
  - [标签](#标签)
  - [重新启动](#重新启动)
  - [Docker Compose 版本](#docker-compose-版本)

<!--
Kompose has support for two providers: OpenShift and Kubernetes.
You can choose a targeted provider using global option `--provider`. If no provider is specified, Kubernetes is set by default.
-->
Kompose 支持 OpenShift 和 Kubernetes 两种提供商，可通过全局选项 `--provider` 指定。如果没有指定，默认选择 Kubernetes。


## `kompose convert`

<!--
Kompose supports conversion of V1, V2, and V3 Docker Compose files into Kubernetes and OpenShift objects.
-->
Kompose 支持将V1, V2, 和 V3版本的 Docker Compose 文件转换成 Kubernetes 和 OpenShift 对象。

### Kubernetes

```sh
$ kompose --file docker-voting.yml convert
WARN Unsupported key networks - ignoring
WARN Unsupported key build - ignoring
INFO Kubernetes file "worker-svc.yaml" created
INFO Kubernetes file "db-svc.yaml" created
INFO Kubernetes file "redis-svc.yaml" created
INFO Kubernetes file "result-svc.yaml" created
INFO Kubernetes file "vote-svc.yaml" created
INFO Kubernetes file "redis-deployment.yaml" created
INFO Kubernetes file "result-deployment.yaml" created
INFO Kubernetes file "vote-deployment.yaml" created
INFO Kubernetes file "worker-deployment.yaml" created
INFO Kubernetes file "db-deployment.yaml" created

$ ls
db-deployment.yaml  docker-compose.yml         docker-gitlab.yml  redis-deployment.yaml  result-deployment.yaml  vote-deployment.yaml  worker-deployment.yaml
db-svc.yaml         docker-voting.yml          redis-svc.yaml     result-svc.yaml        vote-svc.yaml           worker-svc.yaml
```

<!--
You can also provide multiple docker-compose files at the same time:
-->
也可以同时指定多个 docker-compose 文件。

```sh
$ kompose -f docker-compose.yml -f docker-guestbook.yml convert
INFO Kubernetes file "frontend-service.yaml" created         
INFO Kubernetes file "mlbparks-service.yaml" created         
INFO Kubernetes file "mongodb-service.yaml" created          
INFO Kubernetes file "redis-master-service.yaml" created     
INFO Kubernetes file "redis-slave-service.yaml" created      
INFO Kubernetes file "frontend-deployment.yaml" created      
INFO Kubernetes file "mlbparks-deployment.yaml" created      
INFO Kubernetes file "mongodb-deployment.yaml" created       
INFO Kubernetes file "mongodb-claim0-persistentvolumeclaim.yaml" created 
INFO Kubernetes file "redis-master-deployment.yaml" created  
INFO Kubernetes file "redis-slave-deployment.yaml" created   

$ ls
mlbparks-deployment.yaml  mongodb-service.yaml                       redis-slave-service.jsonmlbparks-service.yaml  
frontend-deployment.yaml  mongodb-claim0-persistentvolumeclaim.yaml  redis-master-service.yaml
frontend-service.yaml     mongodb-deployment.yaml                    redis-slave-deployment.yaml
redis-master-deployment.yaml
``` 

<!--
When multiple docker-compose files are provided the configuration is merged. Any configuration that is common will be over ridden by subsequent file.
-->
当指定多个 docker-compose 文件时，配置会被合并。后面的文件会覆盖前面文件中的共有配置项。
 
### OpenShift

```sh
$ kompose --provider openshift --file docker-voting.yml convert
WARN [worker] Service cannot be created because of missing port.
INFO OpenShift file "vote-service.yaml" created             
INFO OpenShift file "db-service.yaml" created               
INFO OpenShift file "redis-service.yaml" created            
INFO OpenShift file "result-service.yaml" created           
INFO OpenShift file "vote-deploymentconfig.yaml" created    
INFO OpenShift file "vote-imagestream.yaml" created         
INFO OpenShift file "worker-deploymentconfig.yaml" created  
INFO OpenShift file "worker-imagestream.yaml" created       
INFO OpenShift file "db-deploymentconfig.yaml" created      
INFO OpenShift file "db-imagestream.yaml" created           
INFO OpenShift file "redis-deploymentconfig.yaml" created   
INFO OpenShift file "redis-imagestream.yaml" created        
INFO OpenShift file "result-deploymentconfig.yaml" created  
INFO OpenShift file "result-imagestream.yaml" created  
```

<!--
It also supports creating buildconfig for build directive in a service. By default, it uses the remote repo for the current git branch as the source repo, and the current branch as the source branch for the build. You can specify a different source repo and branch using ``--build-repo`` and ``--build-branch`` options respectively.
-->
它还支持在一个服务中为 build 指令创建构建配置。它默认使用远程仓库作为当前git分支的源仓库，使用当前分支作为源分支来构建。可以使用 ``--build-repo`` 和 ``--build-branch`` 分别指定源仓库和源分支。

```sh
$ kompose --provider openshift --file buildconfig/docker-compose.yml convert
WARN [foo] Service cannot be created because of missing port. 
INFO OpenShift Buildconfig using git@github.com:rtnpro/kompose.git::master as source. 
INFO OpenShift file "foo-deploymentconfig.yaml" created     
INFO OpenShift file "foo-imagestream.yaml" created          
INFO OpenShift file "foo-buildconfig.yaml" created 
```

<!--
**Note**: If you are manually pushing the Openshift artifacts using ``oc create -f``, you need to ensure that you push the imagestream artifact before the buildconfig artifact, to workaround this Openshift issue: https://github.com/openshift/origin/issues/4518 .
-->
**注意**：如果使用 ``oc create -f`` 手动推送 Openshift，则需要确保在构建配置之前推送 imagestream，以解决此 Openshift 问题：https://github.com/openshift/origin/issues/4518 .

## `kompose up`

<!--
Kompose supports a straightforward way to deploy your "composed" application to Kubernetes or OpenShift via `kompose up`.
-->
Kompose 通过 `kompose up` 命令支持一个简单的方法来部署您的 "composed" 应用程序到 Kubernetes 或者 OpenShift 上。


### Kubernetes
```sh
$ kompose --file ./examples/docker-guestbook.yml up
We are going to create Kubernetes deployments and services for your Dockerized application.
If you need different kind of resources, use the 'kompose convert' and 'kubectl create -f' commands instead.

INFO Successfully created service: redis-master   
INFO Successfully created service: redis-slave    
INFO Successfully created service: frontend       
INFO Successfully created deployment: redis-master
INFO Successfully created deployment: redis-slave
INFO Successfully created deployment: frontend    

Your application has been deployed to Kubernetes. You can run 'kubectl get deployment,svc,pods' for details.

$ kubectl get deployment,svc,pods
NAME                               DESIRED       CURRENT       UP-TO-DATE   AVAILABLE   AGE
deploy/frontend                    1             1             1            1           4m
deploy/redis-master                1             1             1            1           4m
deploy/redis-slave                 1             1             1            1           4m

NAME                               CLUSTER-IP    EXTERNAL-IP   PORT(S)      AGE
svc/frontend                       10.0.174.12   <none>        80/TCP       4m
svc/kubernetes                     10.0.0.1      <none>        443/TCP      13d
svc/redis-master                   10.0.202.43   <none>        6379/TCP     4m
svc/redis-slave                    10.0.1.85     <none>        6379/TCP     4m

NAME                               READY         STATUS        RESTARTS     AGE
po/frontend-2768218532-cs5t5       1/1           Running       0            4m
po/redis-master-1432129712-63jn8   1/1           Running       0            4m
po/redis-slave-2504961300-nve7b    1/1           Running       0            4m
```
<!--
**Note**:
-->
**注意**：

<!--
- You must have a running Kubernetes cluster with a pre-configured kubectl context.
- Only deployments and services are generated and deployed to Kubernetes. If you need different kind of resources, use the `kompose convert` and `kubectl create -f` commands instead.
-->
- 您必须有一个预先配置好 kubectl 的 Kubernetes 集群。
- 仅仅转换为 deployments 和 services 资源并且部署在 Kubernetes。如果您需要不同类型的资源，就使用 `kompose convert` 和 `kubectl create -f` 命令。

### OpenShift
```sh
$ kompose --file ./examples/docker-guestbook.yml --provider openshift up
We are going to create OpenShift DeploymentConfigs and Services for your Dockerized application.
If you need different kind of resources, use the 'kompose convert' and 'oc create -f' commands instead.

INFO Successfully created service: redis-slave    
INFO Successfully created service: frontend       
INFO Successfully created service: redis-master   
INFO Successfully created deployment: redis-slave
INFO Successfully created ImageStream: redis-slave
INFO Successfully created deployment: frontend    
INFO Successfully created ImageStream: frontend   
INFO Successfully created deployment: redis-master
INFO Successfully created ImageStream: redis-master

Your application has been deployed to OpenShift. You can run 'oc get dc,svc,is' for details.

$ oc get dc,svc,is
NAME               REVISION                              DESIRED       CURRENT    TRIGGERED BY
dc/frontend        0                                     1             0          config,image(frontend:v4)
dc/redis-master    0                                     1             0          config,image(redis-master:e2e)
dc/redis-slave     0                                     1             0          config,image(redis-slave:v1)
NAME               CLUSTER-IP                            EXTERNAL-IP   PORT(S)    AGE
svc/frontend       172.30.46.64                          <none>        80/TCP     8s
svc/redis-master   172.30.144.56                         <none>        6379/TCP   8s
svc/redis-slave    172.30.75.245                         <none>        6379/TCP   8s
NAME               DOCKER REPO                           TAGS          UPDATED
is/frontend        172.30.12.200:5000/fff/frontend                     
is/redis-master    172.30.12.200:5000/fff/redis-master                 
is/redis-slave     172.30.12.200:5000/fff/redis-slave    v1  
```

<!--
**Note**:
-->
**注意**：

<!--
- You must have a running OpenShift cluster with a pre-configured `oc`  (`oc login`)
-->
- 您必须有一个预先配置好 `oc`(`oc login`) 的 OpenShift 集群。

## `kompose down`

<!--
Once you have deployed "composed" application to Kubernetes, `$ kompose down` will help you to take the application out by deleting its deployments and services. If you need to remove other resources, use the 'kubectl' command.
-->
一旦部署了 "composed" 应用程序到 Kubernetes，`$ kompose down` 命令将帮助您通过删除它的 deployments 和 services 来将应用程序退出。如果需要删除其他资源，使用 'kubectl' 命令。

```sh
$ kompose --file docker-guestbook.yml down
INFO Successfully deleted service: redis-master   
INFO Successfully deleted deployment: redis-master
INFO Successfully deleted service: redis-slave    
INFO Successfully deleted deployment: redis-slave
INFO Successfully deleted service: frontend       
INFO Successfully deleted deployment: frontend
```

<!--
**Note**:
-->
**注意**

<!--
- You must have a running Kubernetes cluster with a pre-configured kubectl context.
-->
- 您必须有一个预先配置好 kubectl 的 Kubernetes 集群。

<!--
## Build and Push Docker Images
-->
## 构建并推送 Docker 镜像

<!--
Kompose supports both building and pushing Docker images. When using the `build` key within your Docker Compose file, your image will:
-->
Kompose 支持构建和推送 Docker 镜像。当使用 `build` 并指定您的 Docker Compose 文件，您的镜像将会：

<!--
  - Automatically be built with Docker using the `image` key specified within your file
  - Be pushed to the correct Docker repository using local credentials (located at `.docker/config`)
-->
  - 通过 Docker 并根据 `image` 和指定的文件自动构建镜像
  - 使用本地证书（在`.docker/config`）推送镜像到 Docker 仓库
  

<!--
Using an [example Docker Compose file](https://raw.githubusercontent.com/kubernetes/kompose/master/examples/buildconfig/docker-compose.yml):
-->
使用一个[Docker Compose 文件实例](https://raw.githubusercontent.com/kubernetes/kompose/master/examples/buildconfig/docker-compose.yml):

```yaml
version: "2"

services:
    foo:
        build: "./build"
        image: docker.io/foo/bar
```

<!--
Using `kompose up` with a `build` key:
-->
对指定 `build` 键的 Docker Compose 文件，执行 `kompose up` 命令

```none
$ kompose up
INFO Build key detected. Attempting to build and push image 'docker.io/foo/bar' 
INFO Building image 'docker.io/foo/bar' from directory 'build' 
INFO Image 'docker.io/foo/bar' from directory 'build' built successfully 
INFO Pushing image 'foo/bar:latest' to registry 'docker.io' 
INFO Attempting authentication credentials 'https://index.docker.io/v1/ 
INFO Successfully pushed image 'foo/bar:latest' to registry 'docker.io' 
INFO We are going to create Kubernetes Deployments, Services and PersistentVolumeClaims for your Dockerized application. If you need different kind of resources, use the 'kompose convert' and 'kubectl create -f' commands instead. 
 
INFO Deploying application in "default" namespace 
INFO Successfully created Service: foo            
INFO Successfully created Deployment: foo         

Your application has been deployed to Kubernetes. You can run 'kubectl get deployment,svc,pods,pvc' for details.
```

<!--
In order to disable the functionality, or choose to use BuildConfig generation (with OpenShift) `--build (local|build-config|none)` can be passed.
-->
为了禁用该功能，或者通过传入 `--build (local|build-config|none)` 选择使用 BuildConfig 生成（使用 OpenShift）。

```sh
# Disable building/pushing Docker images
$ kompose up --build none

# Generate Build Config artifacts for OpenShift
$ kompose up --provider openshift --build build-config
```

<!--
## Alternative Conversions
-->
## 替代转换

<!--
The default `kompose` transformation will generate Kubernetes [Deployments](/docs/concepts/workloads/controllers/deployment/) and [Services](/docs/concepts/services-networking/service/), in yaml format. You have alternative option to generate json with `-j`. Also, you can alternatively generate [Replication Controllers](/docs/concepts/workloads/controllers/replicationcontroller/) objects, [Daemon Sets](/docs/concepts/workloads/controllers/daemonset/), or [Helm](https://github.com/helm/helm) charts.
-->
默认的 `kompose` 迁移将生成yaml格式的 Kubernetes [Deployments](/docs/concepts/workloads/controllers/deployment/) 和 [Services](/docs/concepts/services-networking/service/),您也可以使用`-j`来选择生成json格式的。此外，您还可以生成
[Replication Controllers](/docs/concepts/workloads/controllers/replicationcontroller/) 对象，[Daemon Sets](/docs/concepts/workloads/controllers/daemonset/)，或者 [Helm](https://github.com/helm/helm)图表。

```sh
$ kompose convert -j
INFO Kubernetes file "redis-svc.json" created
INFO Kubernetes file "web-svc.json" created
INFO Kubernetes file "redis-deployment.json" created
INFO Kubernetes file "web-deployment.json" created
```
<!--
The `*-deployment.json` files contain the Deployment objects.
-->
`*-deployment.json` 文件包含 Deployment 对象。

```sh
$ kompose convert --replication-controller
INFO Kubernetes file "redis-svc.yaml" created
INFO Kubernetes file "web-svc.yaml" created
INFO Kubernetes file "redis-replicationcontroller.yaml" created
INFO Kubernetes file "web-replicationcontroller.yaml" created
```

<!--
The `*-replicationcontroller.yaml` files contain the Replication Controller objects. If you want to specify replicas (default is 1), use `--replicas` flag: `$ kompose convert --replication-controller --replicas 3`
-->
`*-replicationcontroller.yaml`文件包含Replication Controller对象。如果您想要指定副本数（默认是1），使用`--replicas`参数：`$ kompose convert --replication-controller --replicas 3`

```sh
$ kompose convert --daemon-set
INFO Kubernetes file "redis-svc.yaml" created
INFO Kubernetes file "web-svc.yaml" created
INFO Kubernetes file "redis-daemonset.yaml" created
INFO Kubernetes file "web-daemonset.yaml" created
```

<!--
The `*-daemonset.yaml` files contain the Daemon Set objects
-->
`*-daemonset.yaml` 文件包含 Daemon Set 对象。

<!--
If you want to generate a Chart to be used with [Helm](https://github.com/kubernetes/helm) simply do:
-->
如果您想要使用 [Helm](https://github.com/kubernetes/helm) 生成一个图表：

```sh
$ kompose convert -c 
INFO Kubernetes file "web-svc.yaml" created
INFO Kubernetes file "redis-svc.yaml" created
INFO Kubernetes file "web-deployment.yaml" created
INFO Kubernetes file "redis-deployment.yaml" created
chart created in "./docker-compose/"

$ tree docker-compose/
docker-compose
├── Chart.yaml
├── README.md
└── templates
    ├── redis-deployment.yaml
    ├── redis-svc.yaml
    ├── web-deployment.yaml
    └── web-svc.yaml
```

<!--
The chart structure is aimed at providing a skeleton for building your Helm charts.
-->
图表结构是为构建您的 Helm 图表提供一个框架。

<!--
## Labels
-->
## 标签

<!--
`kompose` supports Kompose-specific labels within the `docker-compose.yml` file in order to explicitly define a service's behavior upon conversion.
-->
`kompose` 支持在 `docker-compose.yml` 文件中指定特定的标签，为了在迁移的时候明确地定义一个服务的行为。

<!--
- `kompose.service.type` defines the type of service to be created.
-->
- `kompose.service.type` 定义了将被创建服务的类型。

For example:

```yaml
version: "2"
services: 
  nginx:
    image: nginx
    dockerfile: foobar
    build: ./foobar
    cap_add:
      - ALL
    container_name: foobar
    labels: 
      kompose.service.type: nodeport
```

<!--
- `kompose.service.expose` defines if the service needs to be made accessible from outside the cluster or not. If the value is set to "true", the provider sets the endpoint automatically, and for any other value, the value is set as the hostname. If multiple ports are defined in a service, the first one is chosen to be the exposed.
  - For the Kubernetes provider, an ingress resource is created and it is assumed that an ingress controller has already been configured.
  - For the OpenShift provider, a route is created.
-->
- `kompose.service.expose` 定义了是否这个服务能从集群外部来访问。如果设置为"true"，提供者（例如 Kubernetes）自动设置服务的 endpoint，对于任何其他值，该值被设置为主机名。如果一个服务定义了多个端口，
选择暴漏第一个。
  - 对 Kubernetes 来说，创建了一个 ingress 资源，假设 ingress controller 已经被配置。
  - 对 OpenShift 来说，路由已经被创建。

<!--
For example:
-->
例如：

```yaml
version: "2"
services:
  web:
    image: tuna/docker-counter23
    ports:
     - "5000:5000"
    links:
     - redis
    labels:
      kompose.service.expose: "counter.example.com"
  redis:
    image: redis:3.0
    ports:
     - "6379"
```

<!--
The currently supported options are:
-->
当前支持的选项有：

| Key                  | Value                               |
|----------------------|-------------------------------------|
| kompose.service.type | nodeport / clusterip / loadbalancer |
| kompose.service.expose| true / hostname |

<!--
**Note**: `kompose.service.type` label should be defined with `ports` only, otherwise `kompose` will fail.
-->
**注意**： `kompose.service.type` 标签应该只被定义为 `ports`，否则，`kompose` 将会失败。

<!--
## Restart
-->
## 重新启动

<!--
If you want to create normal pods without controllers you can use `restart` construct of docker-compose to define that. Follow table below to see what happens on the `restart` value.
-->
如果您想要创建没有控制器的普通 pods，可以使用 docker-compose 的 `restart` 来定义它。请按照下面的表格查看 `restart` 值变化会发生什么。

| `docker-compose` `restart` | object created    | Pod `restartPolicy` |
|----------------------------|-------------------|---------------------|
| `""`                       | controller object | `Always`            |
| `always`                   | controller object | `Always`            |
| `on-failure`               | Pod               | `OnFailure`         |
| `no`                       | Pod               | `Never`             |

<!--
**Note**: controller object could be `deployment` or `replicationcontroller`, etc.
-->
**注意**： 控制器对象可以是 `deployment` 或者 `replicationcontroller`，等。

<!--
For e.g. `pival` service will become pod down here. This container calculated value of `pi`.
-->
For e.g. `pival` 服务将会变为普通 pod。容器中计算的值为 `pi`。

```yaml
version: '2'

services:
  pival:
    image: perl
    command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
    restart: "on-failure"
```

<!--
### Warning about Deployment Config's
-->
### Deployment 配置的警告

<!--
If the Docker Compose file has a volume specified for a service, the Deployment (Kubernetes) or DeploymentConfig (OpenShift) strategy is changed to "Recreate" instead of "RollingUpdate" (default). This is done to avoid multiple instances of a service from accessing a volume at the same time.
-->
如果 Docker Compose 文件有一个服务指定的卷，则 Deployment（Kubernetes） 或者 DeploymentConfig（OpenShift）的策略改为 "Recreate" 而不是 "RollingUpdate"（默认的）。这样做是为了避免服务的多个实例同时访问卷。

<!--
If the Docker Compose file has service name with `_` in it (eg.`web_service`), then it will be replaced by `-` and the service name will be renamed accordingly (eg.`web-service`). Kompose does this because "Kubernetes" doesn't allow `_` in object name.
-->
如果 Docker Compose 文件中有带`_`的服务名（例如：`web_service`），那么它将被`-`替换，并且服务名将相应地重命名(例如：`web-service`)。Kompose 之所以这样做是因为 "Kubernetes" 的对象名不允许`_`。

<!--
Please note that changing service name might break some `docker-compose` files.
-->
请注意，更改服务名称可能会破坏一些 `docker-compose` 文件。

<!--
## Docker Compose Versions
-->
## Docker Compose 版本

<!--
Kompose supports Docker Compose versions: 1, 2 and 3. We have limited support on versions 2.1 and 3.2 due to their experimental nature.
-->
Kompose 支持 Docker Compose 的1,2和3版本。由于实验的性质，我们对于2.1和3.2版本的支持有限。

<!--
A full list on compatibility between all three versions is listed in our [conversion document](https://github.com/kubernetes/kompose/blob/master/docs/conversion.md) including a list of all incompatible Docker Compose keys.
-->
在我们的转换文档 [conversion document](https://github.com/kubernetes/kompose/blob/master/docs/conversion.md)中列出了三个版本之间的兼容性列表，其中包括所有不兼容 Docker Compose Keys的列表。

{{% /capture %}}