---
---
我们有很多不同类型的参考案例可供选择：

* 『集群应用示例』和『持久化数据卷示例』展示了如何部署当下流行的软件包，例如OpenShift、Ceph、Cassandra和WordPress。这些案例尝试告诉你如何在Kubernetes上使用那些你已经熟悉的软件包。

* 『多层级的应用软件示例』则是一个出于教学目而构造的例子，突出强调使用Kubernetes部署应用需要了解的关键概念，以及构建适合于Kubernetes的应用的基本原则。


## 最近更新的应用

Kubetentes小组将这些应用示例标记为最近更新的。它们在不久前刚刚被检查过，因此可以确保其内容质量良好且能够在Kubernetes {{page.version}}版本中正常使用。

<p>&nbsp;</p>

<style>
h2, h3, h4 {
  border-bottom: 0px !important;
}
.colContainer {
  padding-top:2px;
  padding-left: 2px;
  overflow: auto;
}
#samples a {
  color: #000;
}
.col3rd {
  display: block;
  width: 250px;
  float: left;
  margin-right: 30px;
  margin-bottom: 30px;
}
.col3rd h3, .col2nd h3 {
  margin-bottom: 0px !important;
}
.col3rd .button, .col2nd .button {
  margin-top: 20px;
  border-radius: 2px;
}
.col3rd p, .col2nd p {
  margin-left: 2px;
}
.col2nd {
  display: block;
  width: 400px;
  float: left;
  margin-right: 30px;
  margin-bottom: 30px;
}
.shadowbox {
  display: inline;
  float: left;
  text-transform: none;
  font-weight: bold;
  text-align: center;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
  line-height: 24px;
  position: relative;
  display: block;
  cursor: pointer;
  box-shadow: 0 2px 2px rgba(0,0,0,.24),0 0 2px rgba(0,0,0,.12);
  border-radius: 10px;
  background: #fff;
  transition: all .3s;
  padding: 16px;
  margin: 0 16px 16px 0;
  text-decoration: none;
  letter-spacing: .01em;
}
.shadowbox img {
    min-width: 150px;
    max-width: 150px;
    max-height: 50px;
}
</style>

<div id="samples" class="colContainer">
<a href="/docs/getting-started-guides/meanstack/" class="shadowbox">
  <img src="/images/docs/meanstack/image_0.png"><br/>MEAN Stack
</a>
<a href="https://github.com/kubernetes/kubernetes/tree/{{page.githubbranch}}/examples/guestbook" target="_blank" class="shadowbox">
  <img src="/images/docs/redis.svg"><br/>Guestbook + Redis
</a>
<a href="https://github.com/kubernetes/kubernetes/tree/{{page.githubbranch}}/examples/cassandra" target="_blank" class="shadowbox">
  <img src="/images/docs/cassandra.svg"><br/>Cloud Native Cassandra
</a>
<a href="https://github.com/kubernetes/kubernetes/tree/{{page.githubbranch}}/examples/mysql-wordpress-pd/" target="_blank" class="shadowbox">
  <img src="/images/docs/wordpress.svg"><br/>WordPress + MySQL
</a>
</div>
