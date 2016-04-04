---
---
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
  overflow: hidden;
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
  overflow: hidden;
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
<div class="colContainer">
  <div class="col3rd">
    <h3>Kubernetes是什么？</h3>
    <p>Kubernetes是一个支持自动部署、水平扩容以及跨集群应用容器运维的开源平台。下面这个链接更详细的介绍了这个平台能为你带来什么。</p>
    <a href="/docs/whatisk8s/" class="button">阅读Kubernetes简介</a>
  </div>
  <div class="col3rd">
    <h3>快速入门</h3>
    <p>我们会用Docker在你的机器上创建一个Kubernetes实例，并在上面运行一个简单的Node.js "Hello World"应用。只需要5分钟，你就可以得到一个可部署的应用。</p>
    <a href="/docs/hellonode/" class="button">让我们开始吧</a>
  </div>
  <div class="col3rd">
    <h3>入门引导</h3>
    <p>如果你已经完成了前面那个简单的例子，强烈推荐你继续阅读下面这个『Kubernetes 101』案例。 这个案例中包含了Kubernetes许多重要的功能，并通过实际的代码展示其中涉及的关键概念。 除此之外，还有一个<a href="/docs/user-guide/walkthrough/k8s201">『Kubernetes 201』</a>案例！</p>
    <a href="/docs/user-guide/walkthrough/" class="button">『Kubernetes 101』案例</a>
  </div>
</div>

## 更多例子

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

<p>&nbsp;</p>
<p>&nbsp;</p>

<div class="colContainer">
  <div class="col2nd">
  <h3>参与贡献文档</h3>
  <p>与Kubernetes的代码一样，Kubernetes的文档也是开源的。在GitHub上可以找到这些文档的源代码，你可以直接Fork它，只需要简单的配置，就可以通过『你的GitHub账号名.github.io』这个网址来快速查看你的修改了！</p>
  <a href="/editdocs/" class="button">告诉我如何贡献文档</a>
  </div>
  <div class="col2nd">
  <h3>需要帮助？</h3>
  <p>尝试在我们的<a href="/docs/troubleshooting/">错误排查指南</a>或者<a href="https://github.com/kubernetes/kubernetes/wiki/User-FAQ">常见问题列表</a>中查找相应的内容。此外，Kubernetes有很多社区贡献者和专家常常出没在我们的<a href="http://slack.kubernetes.io/">Slack channel</a>、<a href="https://groups.google.com/forum/#!forum/google-containers">Google Group</a>以及<a href="http://stackoverflow.com/questions/tagged/kubernetes">Stack Overflow</a>网站上。</p>
  <a href="/docs/troubleshooting/" class="button">告诉我如何寻求帮助</a>
  </div>
</div>
