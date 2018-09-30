---
reviewers:
- erictune
- lavalamp
- ericchiang
- deads2k
- liggitt
title: Authenticating
content_template: templates/concept
weight: 10
---

{{% capture overview %}}
<!-- This page provides an overview of authenticating. -->
此页面提供了身份验证(authc)的概述。
{{% /capture %}}

{{% capture body %}}
<!-- ## Users in Kubernetes -->
## Kubernetes 中的用户

<!-- All Kubernetes clusters have two categories of users: service accounts managed
by Kubernetes, and normal users.-->
所有 Kubernetes 集群都有两类用户：由Kubernetes管理的服务帐户和普通用户。

<!-- Normal users are assumed to be managed by an outside, independent service. An
admin distributing private keys, a user store like Keystone or Google Accounts,
even a file with a list of usernames and passwords. In this regard, _Kubernetes
does not have objects which represent normal user accounts._ Regular users
cannot be added to a cluster through an API call. -->
假设普通用户由外部独立服务管理。一个管理员分发私钥，像 Keystone 或 Google帐户 这样的用户商店，
甚至包含用户名和密码列表的文件。
在这方面，_Kubernetes没有代表普通用户帐户的对象._ 常规用户无法通过API调用添加到群集。

<!-- In contrast, service accounts are users managed by the Kubernetes API. They are
bound to specific namespaces, and created automatically by the API server or
manually through API calls. Service accounts are tied to a set of credentials
stored as `Secrets`, which are mounted into pods allowing in-cluster processes
to talk to the Kubernetes API. -->
相反，服务帐户是由 Kubernetes API 管理的用户。
他们是绑定到特定命名空间，并由API服务器自动创建或者通过手动调用 API。
服务帐户与一组凭据相关联存储为 “Secrets”，它们被挂载到pod中，允许集群内进程与 Kubernetes API 交互。

<!-- API requests are tied to either a normal user or a service account, or are treated
as anonymous requests. This means every process inside or outside the cluster, from
a human user typing `kubectl` on a workstation, to `kubelets` on nodes, to members
of the control plane, must authenticate when making requests to the API server,
or be treated as an anonymous user. -->
API请求与普通用户或服务帐户绑定，或者作为匿名请求被处理。
这意味着集群内部或外部的每个过程，从用户在工作站上输入`kubectl`，到在节点上输入`kubelets`，到成员
控制面板，在向 API 服务器发出请求时都必须进行身份验证，否则被视为匿名用户。

<!-- ## Authentication strategies -->
## Authentication 策略

<!-- Kubernetes uses client certificates, bearer tokens, an authenticating proxy, or HTTP basic auth to
authenticate API requests through authentication plugins. As HTTP requests are
made to the API server, plugins attempt to associate the following attributes
with the request: -->
Kubernetes 使用客户端证书，bearer token，身份验证代理或 HTTP 基本身份验证等方式通过身份验证插件验证API请求。
当 HTTP 请求发送到API服务器时，插件尝试关联请求的以下属性：

<!-- 
* Username: a string which identifies the end user. Common values might be `kube-admin` or `jane@example.com`.
* UID: a string which identifies the end user and attempts to be more consistent and unique than username.
* Groups: a set of strings which associate users with a set of commonly grouped users.
* Extra fields: a map of strings to list of strings which holds additional information authorizers may find useful. 
-->
* Username: 标识最终用户的字符串。常见的值可能是`kube-admin`或`jane@example.com`。
* UID: 一个字符串，用于标识最终用户并尝试比用户名更加一致和唯一。
* Groups: 一组字符串，用于将用户与同一组分组的用户相关联。
* Extra fields: map 列表，其中包含授权者可能认为有用的附加信息。

<!-- All values are opaque to the authentication system and only hold significance
when interpreted by an [authorizer](/docs/admin/authorization/). -->
所有值对身份验证系统都是不透明的，除非其含义被[authorizer](/docs/admin/authorization/)解释。

<!-- You can enable multiple authentication methods at once. You should usually use at least two methods: -->
您可以一次启用多个身份验证方法。您通常应该使用至少两种方法：

<!--
 - service account tokens for service accounts
 - at least one other method for user authentication.
 -->
 - 服务帐户的服务帐户令牌 
 - 至少一种用于用户身份验证的方法。

<!-- When multiple authenticator modules are enabled, the first module
to successfully authenticate the request short-circuits evaluation.
The API server does not guarantee the order authenticators run in. -->
启用多个身份验证器模块时，第一个模块成功验证时请求短路求值。
API服务器不保证验证器运行的顺序。

<!-- The `system:authenticated` group is included in the list of groups for all authenticated users. -->
`system：authenticated`组包含在所有经过身份验证的用户。

<!-- Integrations with other authentication protocols (LDAP, SAML, Kerberos, alternate x509 schemes, etc)
can be accomplished using an [authenticating proxy](#authenticating-proxy) or the
[authentication webhook](#webhook-token-authentication). -->
与其他身份验证协议（LDAP，SAML，Kerberos，备用x509方案等）的集成
可以使用[身份验证代理](＃authenticating-proxy)或[authentication webhook](#webhook-token-authentication)。

<!-- ### X509 Client Certs -->
### X509 客户端证书

<!-- Client certificate authentication is enabled by passing the `--client-ca-file=SOMEFILE`
option to API server. The referenced file must contain one or more certificates authorities
to use to validate client certificates presented to the API server. If a client certificate
is presented and verified, the common name of the subject is used as the user name for the
request. As of Kubernetes 1.4, client certificates can also indicate a user's group memberships
using the certificate's organization fields. To include multiple group memberships for a user,
include multiple organization fields in the certificate. -->
通过传递`--client-ca-file=SOMEFILE`来启用API服务器客户端证书身份验证选项。
引用的文件必须包含一个或多个证书颁发机构用于验证提供给API服务器的客户端证书。
如果一个客户证书被呈递和被验证，主题的通用名称用作用户名进行请求。
从Kubernetes 1.4开始，客户端证书也可以使用证书的组织(org)字段指示用户的组成员身份。
要为用户包含多个组成员身份，在证书中包含多个组织字段。

<!-- For example, using the `openssl` command line tool to generate a certificate signing request: -->
例如，使用`openssl`命令行工具生成证书签名请求：

``` bash
openssl req -new -key jbeda.pem -out jbeda-csr.pem -subj "/CN=jbeda/O=app1/O=app2"
```

<!-- This would create a CSR for the username "jbeda", belonging to two groups, "app1" and "app2". -->
这将为用户名“jbeda”创建CSR，属于两个组“app1”和“app2”。

<!-- See [Managing Certificates](/docs/concepts/cluster-administration/certificates/) for how to generate a client cert. -->
有关如何生成客户端证书，请参阅[管理证书](/docs/concepts/cluster-administration/certificates/)。

<!-- ### Static Token File -->
### 静态 token 文件

<!-- The API server reads bearer tokens from a file when given the `--token-auth-file=SOMEFILE` option on the command line.  Currently, tokens last indefinitely, and the token list cannot be
changed without restarting API server. -->
当在命令行上给出`--token-auth-file=SOMEFILE`选项时，API服务器从文件中读取 bearer token。
目前，令牌无限期地持续，并且令牌列表不能更改,除非重新启动API服务器。

<!-- The token file is a csv file with a minimum of 3 columns: token, user name, user uid,
followed by optional group names. -->
令牌文件是一个至少包含3列的csv文件：令牌，用户名，用户uid，后跟可选的组名。

{{< note >}}
<!-- **Note:** If you have more than one group the column must be double quoted e.g. -->
**注意:** 如果您有多个组，则该列必须加双引号，例如：

```conf
token,user,uid,"group1,group2,group3"
```
{{< /note >}}

<!-- #### Putting a Bearer Token in a Request -->
#### 将bearer-token放入请求中

<!-- When using bearer token authentication from an http client, the API
server expects an `Authorization` header with a value of `Bearer
THETOKEN`.  The bearer token must be a character sequence that can be
put in an HTTP header value using no more than the encoding and
quoting facilities of HTTP.  For example: if the bearer token is
`31ada4fd-adec-460c-809a-9e56ceb75269` then it would appear in an HTTP
header as shown below. -->
从http客户端使用承载令牌身份验证时，API服务器需要一个`Authorization`请求头，其值为`Beare THETOKEN`。
承载令牌必须是一个字符序列，只需使用HTTP的编码和引用功能就可以将其置于HTTP头值中。
例如：如果承载令牌是`31ada4fd-adec-460c-809a-9e56ceb75269`那么它会出现在HTTP头中，如下图所示:

```http
Authorization: Bearer 31ada4fd-adec-460c-809a-9e56ceb75269
```

<!-- ### Bootstrap Tokens -->
### Bootstrap 令牌

<!-- This feature is currently in **alpha**. -->
此功能目前处于 **alpha** 状态。

<!--To allow for streamlined bootstrapping for new clusters, Kubernetes includes a
dynamically-managed Bearer token type called a *Bootstrap Token*. These tokens
are stored as Secrets in the `kube-system` namespace, where they can be
dynamically managed and created. Controller Manager contains a TokenCleaner
controller that deletes bootstrap tokens as they expire. -->
为了实现新集群的简化引导，Kubernetes包括一个动态管理的承载令牌类型称为 *Bootstrap Token* 。
这些令牌在`kube-system`命名空间中存储为Secrets，它们可以是动态管理和创建。
Controller Manager包含TokenCleaner控制器在过期时删除 Bootstrap 令牌。

<!-- The tokens are of the form `[a-z0-9]{6}.[a-z0-9]{16}`.  The first component is a
Token ID and the second component is the Token Secret.  You specify the token
in an HTTP header as follows: -->
标记的形式为`[a-z0-9]{6}.[a-z0-9]{16}`。
第一个组成部分是令牌ID和第二个组成部分是令牌密钥。
您在HTTP标头中如下指定令牌：

```http
Authorization: Bearer 781292.db7bc3a58fc5f07e
```

<!-- You must enable the Bootstrap Token Authenticator with the
`--experimental-bootstrap-token-auth` flag on the API Server.  You must enable
the TokenCleaner controller via the `--controllers` flag on the Controller
Manager.  This is done with something like `--controllers=*,tokencleaner`.
`kubeadm` will do this for you if you are using it to bootstrapping a cluster. -->
您必须API服务器上的`--experimental-bootstrap-token-auth`标志来启用Bootstrap Token Authenticator 。
你必须通过Controller上的`--controllers`标志管理来启用 TokenCleaner 控制器。
这是通过`--controllers=*，tokencleaner`之类的东西来完成的。
如果您使用它来引导群集，`kubeadm`将为您执行此操作。

<!-- The authenticator authenticates as `system:bootstrap:<Token ID>`.  It is
included in the `system:bootstrappers` group.  The naming and groups are
intentionally limited to discourage users from using these tokens past
bootstrapping.  The user names and group can be used (and are used by `kubeadm`)
to craft the appropriate authorization policies to support bootstrapping a
cluster. -->
验证器验证为`system：bootstrap:<Token ID>`。它是包含在`system:bootstrappers`组中。
命名和组是故意限制以阻止用户过去使用这些令牌引导。
用户名和组可以使用（由`kubeadm`使用）来制定适当的授权策略以支持引导一个集群。

<!-- Please see [Bootstrap Tokens](/docs/admin/bootstrap-tokens/) for in depth
documentation on the Bootstrap Token authenticator and controllers along with
how to manage these tokens with `kubeadm`. -->
请深入了解[Bootstrap Tokens](/docs/admin/bootstrap-tokens/)Bootstrap令牌验证器 和 控制器 文档以了解如何使用`kubeadm`管理这些令牌。

<!-- ### Static Password File -->
### 静态密码文件

<!-- Basic authentication is enabled by passing the `--basic-auth-file=SOMEFILE`
option to API server. Currently, the basic auth credentials last indefinitely,
and the password cannot be changed without restarting API server. Note that basic
authentication is currently supported for convenience while we finish making the
more secure modes described above easier to use. -->
通过传递`--basic-auth-file=SOMEFILE`来启用API服务器的基本身份验证选项。
目前，基本身份验证凭证无限期地持续，如果不重新启动API服务器，则无法更改密码。
请注意，为方便起见，目前支持基本身份验证，同时我们完成了上述更安全的模式，更易于使用。

<!-- The basic auth file is a csv file with a minimum of 3 columns: password, user name, user id.
In Kubernetes version 1.6 and later, you can specify an optional fourth column containing
comma-separated group names. If you have more than one group, you must enclose the fourth
column value in double quotes ("). See the following example: -->
基本auth文件是一个至少包含3列的csv文件：密码，用户名，用户ID。
在Kubernetes 1.6及更高版本中，您可以指定包含的可选第四列 逗号分隔的组名。
如果您有多个组，则必须使用双引号（"）包含第四个组中的列值。请参阅以下示例：

```conf
password,user,uid,"group1,group2,group3"
```

<!-- When using basic authentication from an http client, the API server expects an `Authorization` header
with a value of `Basic BASE64ENCODED(USER:PASSWORD)`. -->
从http客户端使用基本身份验证时，API服务器需要`Authorization`标头
值为`Basic BASE64ENCODED(USER:PASSWORD)`。

<!-- ### Service Account Tokens -->
### 服务帐户令牌

<!-- A service account is an automatically enabled authenticator that uses signed
bearer tokens to verify requests. The plugin takes two optional flags: -->
服务帐户是一个自动启用的身份验证器，它使用签名的承载令牌来验证请求。该插件有两个可选标志：

<!-- * `--service-account-key-file` A file containing a PEM encoded key for signing bearer tokens.
If unspecified, the API server's TLS private key will be used.
* `--service-account-lookup` If enabled, tokens which are deleted from the API will be revoked. -->
*`--service-account-key-file`包含用于签署承载令牌的PEM编码密钥的文件。如果未指定，将使用API​​服务器的TLS私钥。
*`--service-account-lookup`如果启用，从API中删除的令牌将被撤销。

<!-- Service accounts are usually created automatically by the API server and
associated with pods running in the cluster through the `ServiceAccount`
[Admission Controller](/docs/admin/admission-controllers/). Bearer tokens are
mounted into pods at well-known locations, and allow in-cluster processes to
talk to the API server. Accounts may be explicitly associated with pods using the
`serviceAccountName` field of a `PodSpec`. -->
服务帐户通常由API服务器自动创建，并通过`ServiceAccount`与群集中运行的pod相关联 [准入控制](/docs/admin/admission-controllers/). 
承载令牌安装在众所周知的位置的pod中，并允许集群内进程与API服务器通信。可以使用`PodSpec`的`serviceAccountName`字段将帐户与pod明确关联。

{{< note >}}
<!-- **Note:** `serviceAccountName` is usually omitted because this is done automatically. -->
**注意:** `serviceAccountName`通常被省略，因为这是自动完成的。
{{< /note >}}

```yaml
apiVersion: apps/v1 # this apiVersion is relevant as of Kubernetes 1.9
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: default
spec:
  replicas: 3
  template:
    metadata:
    # ...
    spec:
      serviceAccountName: bob-the-bot
      containers:
      - name: nginx
        image: nginx:1.7.9
```

<!-- Service account bearer tokens are perfectly valid to use outside the cluster and
can be used to create identities for long standing jobs that wish to talk to the
Kubernetes API. To manually create a service account, simply use the `kubectl
create serviceaccount (NAME)` command. This creates a service account in the
current namespace and an associated secret. -->
服务帐户承载令牌完全有效，可在群集外部使用，并可用于为希望与Kubernetes API通信的长期工作创建身份。
要手动创建服务帐户，只需使用`kubectl create serviceaccount（NAME）`命令。
这将在当前名称空间中创建一个服务帐户以及一个相关的 secret。

```
$ kubectl create serviceaccount jenkins
serviceaccount "jenkins" created
$ kubectl get serviceaccounts jenkins -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  # ...
secrets:
- name: jenkins-token-1yvwg
```

<!-- The created secret holds the public CA of the API server and a signed JSON Web
Token (JWT).-->
创建的 secret 包含API服务器的公共CA和签名的JSON Web Token(JWT)。

```
$ kubectl get secret jenkins-token-1yvwg -o yaml
apiVersion: v1
data:
  ca.crt: (APISERVER'S CA BASE64 ENCODED)
  namespace: ZGVmYXVsdA==
  token: (BEARER TOKEN BASE64 ENCODED)
kind: Secret
metadata:
  # ...
type: kubernetes.io/service-account-token
```

{{< note >}}
<!-- **Note:** Values are base64 encoded because secrets are always base64 encoded. -->
**注意:** 值是base64编码的，因为 secret 始终是 base64 编码的。
{{< /note >}}

<!-- The signed JWT can be used as a bearer token to authenticate as the given service
account. See [above](#putting-a-bearer-token-in-a-request) for how the token is included
in a request.  Normally these secrets are mounted into pods for in-cluster access to
the API server, but can be used from outside the cluster as well. -->
签名的JWT可以用作承载令牌，以作为给定的服务帐户进行身份验证。
请参阅[上面]（＃puts-a-bearer-token-in-a-request），了解令牌如何包含在请求中。
通常，这些秘密会安装到pod中，以便在群集中访问API服务器，但也可以在群集外部使用。

<!-- Service accounts authenticate with the username `system:serviceaccount:(NAMESPACE):(SERVICEACCOUNT)`,
and are assigned to the groups `system:serviceaccounts` and `system:serviceaccounts:(NAMESPACE)`. -->
服务帐户使用用户名`system:serviceaccount:(NAMESPACE):(SERVICEACCOUNT)`进行身份验证，
并分配给`system:serviceaccounts`和`system:serviceaccounts:(NAMESPACE)`组。

<!-- WARNING: Because service account tokens are stored in secrets, any user with
read access to those secrets can authenticate as the service account. Be cautious
when granting permissions to service accounts and read capabilities for secrets. -->
警告：因为服务帐户令牌存储在机密中，所以任何对这些机密的读访问权的用户都有可以作为服务帐户进行身份验证。
授予服务帐户权限和读取秘密功能时要小心。

<!-- ### OpenID Connect Tokens -->
### OpenID Connect Tokens

<!-- [OpenID Connect](https://openid.net/connect/) is a flavor of OAuth2 supported by
some OAuth2 providers, notably Azure Active Directory, Salesforce, and Google.
The protocol's main extension of OAuth2 is an additional field returned with
the access token called an [ID Token](https://openid.net/specs/openid-connect-core-1_0.html#IDToken).
This token is a JSON Web Token (JWT) with well known fields, such as a user's
email, signed by the server. -->
[OpenID Connect](https://openid.net/connect/)是OAuth2的一种风格，由一些OAuth2提供商支持，特别是Azure Active Directory，Salesforce和Google。
该协议的OAuth2主要扩展是一个附加字段，其中包含名为[ID Token](https://openid.net/specs/openid-connect-core-1_0.html#IDToken)的访问令牌。
此令牌是JSON Web令牌（JWT），具有由服务器签名的众所周知的字段，例如用户的电子邮件。

<!-- To identify the user, the authenticator uses the `id_token` (not the `access_token`)
from the OAuth2 [token response](https://openid.net/specs/openid-connect-core-1_0.html#TokenResponse)
as a bearer token.  See [above](#putting-a-bearer-token-in-a-request) for how the token
is included in a request. -->
为了识别用户，验证者使用来自 OAuth2 [令牌响应](https://openid.net/specs/openid-connect-core-1_0.html#TokenResponse)的`id_token`（不是`access_token`）作为承载令牌。
请参阅[上面](#将bearer-token放入请求中)，了解令牌如何包含在请求中。

![Kubernetes OpenID Connect Flow](/images/docs/admin/k8s_oidc_login.svg)

<!--
1.  Login to your identity provider
2.  Your identity provider will provide you with an `access_token`, `id_token` and a `refresh_token`
3.  When using `kubectl`, use your `id_token` with the `--token` flag or add it directly to your `kubeconfig`
4.  `kubectl` sends your `id_token` in a header called Authorization to the API server
5.  The API server will make sure the JWT signature is valid by checking against the certificate named in the configuration
6.  Check to make sure the `id_token` hasn't expired
7.  Make sure the user is authorized
8.  Once authorized the API server returns a response to `kubectl`
9.  `kubectl` provides feedback to the user 
-->
1. 登录您的身份提供商
2. 您的身份提供者将为您提供`access_token`，`id_token`和`refresh_token`
3. 使用`kubectl`时，请将`id_token`与`--token`标志一起使用，或将其直接添加到`kubeconfig`中。
4. `kubectl`将名为Authorization的头中的`id_token`发送到API服务器
5. API服务器将通过检查配置中指定的证书来确保JWT签名有效
6. 检查以确保`id_token`没有过期
7. 确保用户已获得授权
8. 授权后，API服务器返回对`kubectl`的响应
9. `kubectl`向用户提供反馈

<!-- Since all of the data needed to validate who you are is in the `id_token`, Kubernetes doesn't need to
"phone home" to the identity provider.  In a model where every request is stateless this provides a very scalable
solution for authentication.  It does offer a few challenges: -->
由于验证您的身份所需的所有数据都在`id_token`中，因此Kubernetes不需要向身份提供者“打电话回家”。
在每个请求都是无状态的模型中，这为身份验证提供了非常可扩展的解决方案。它确实提供了一些挑战：

<!--
1.  Kubernetes has no "web interface" to trigger the authentication process.  There is no browser or interface to collect credentials which is why you need to authenticate to your identity provider first.
2.  The `id_token` can't be revoked, it's like a certificate so it should be short-lived (only a few minutes) so it can be very annoying to have to get a new token every few minutes.
3.  There's no easy way to authenticate to the Kubernetes dashboard without using the `kubectl proxy` command or a reverse proxy that injects the `id_token`.
-->
1. Kubernetes没有“Web界面”来触发身份验证过程。没有用于收集凭据的浏览器或界面，这就是您需要首先向身份提供商进行身份验证的原因。
2. 不能撤销`id_token`，它就像一个证书，所以它应该是短暂的（只有几分钟），所以每隔几分钟就得到一个新的令牌会非常烦人。
3. 没有使用`kubectl proxy`命令或注入`id_token`的反向代理，没有简单的方法来验证Kubernetes仪表板。

<!-- #### Configuring the API Server -->
#### 配置API服务器

<!-- To enable the plugin, configure the following flags on the API server: -->
要启用插件，请在API服务器上配置以下标志：

<!--
| Parameter | Description | Example | Required |
| --------- | ----------- | ------- | ------- |
| `--oidc-issuer-url` | URL of the provider which allows the API server to discover public signing keys. Only URLs which use the `https://` scheme are accepted.  This is typically the provider's discovery URL without a path, for example "https://accounts.google.com" or "https://login.salesforce.com".  This URL should point to the level below .well-known/openid-configuration | If the discovery URL is `https://accounts.google.com/.well-known/openid-configuration`, the value should be `https://accounts.google.com` | Yes |
| `--oidc-client-id` |  A client id that all tokens must be issued for. | kubernetes | Yes |
| `--oidc-username-claim` | JWT claim to use as the user name. By default `sub`, which is expected to be a unique identifier of the end user. Admins can choose other claims, such as `email` or `name`, depending on their provider. However, claims other than `email` will be prefixed with the issuer URL to prevent naming clashes with other plugins. | sub | No |
| `--oidc-username-prefix` | Prefix prepended to username claims to prevent clashes with existing names (such as `system:` users). For example, the value `oidc:` will create usernames like `oidc:jane.doe`. If this flag isn't provided and `--oidc-user-claim` is a value other than `email` the prefix defaults to `( Issuer URL )#` where `( Issuer URL )` is the value of `--oidc-issuer-url`. The value `-` can be used to disable all prefixing. | `oidc:` | No |
| `--oidc-groups-claim` | JWT claim to use as the user's group. If the claim is present it must be an array of strings. | groups | No |
| `--oidc-groups-prefix` | Prefix prepended to group claims to prevent clashes with existing names (such as `system:` groups). For example, the value `oidc:` will create group names like `oidc:engineering` and `oidc:infra`. | `oidc:` | No |
| `--oidc-ca-file` | The path to the certificate for the CA that signed your identity provider's web certificate.  Defaults to the host's root CAs. | `/etc/kubernetes/ssl/kc-ca.pem` | No |
-->

| 参数 | 描述 | 示例 | 必要 |
| --------- | ----------- | ------- | ------- |
| `--oidc-issuer-url` | 允许API服务器发现公共签名密钥的提供程序的URL。只接受使用`https：//`方案的URL。这通常是提供者的没有路径的发现URL，例如"https://accounts.google.com"或"https://login.salesforce.com"。此URL应指向以下级别 .well-known/openid-configuration | 如果发现网址为"https://accounts.google.com/.well-known/openid-configuration"，则该值应为"https://accounts.google.com"|yes|
| `--oidc-client-id` |  必须为其颁发所有令牌的客户端ID | kubernetes |yes|
| `--oidc-username-claim` | JWT声明用作用户名。默认为`sub`，预计是最终用户的唯一标识符。管理员可以选择其他声明，例如“email”或“name”，具体取决于其提供商。但是，"email"以外的声明将以发行者URL为前缀，以防止与其他插件发生冲突。 | sub |no|
| `--oidc-username-prefix` | Prefix附加在用户名声明之前，以防止与现有名称冲突（例如`system:`users）。例如，值"oidc:"将创建像`oidc:jane.doe`这样的用户名。如果没有提供这个标志，并且`--oidc-user-claim`是除`email`之外的值，则前缀默认为`(Issuer URL)#`其中`(Issuer URL)`是`--oidc-issuer-url`的值，值`-`可用于禁用所有前缀。 | `oidc:`|no|
| `--oidc-groups-claim` | JWT声称用作用户组。如果声明存在，则它必须是字符串数组。 | groups | No |
| `--oidc-groups-prefix` | 前缀用于对声明进行分组以防止与现有名称冲突（例如`system:`groups）。例如，值`oidc:`将创建组名，如`oidc:engineering`和`oidc:infra`。| `oidc:` | No |
| `--oidc-ca-file` |签署身份提供商的Web证书的CA证书的路径。默认为主机的根CA. | `/etc/kubernetes/ssl/kc-ca.pem` | No |
<!--
Importantly, the API server is not an OAuth2 client, rather it can only be
configured to trust a single issuer. This allows the use of public providers,
such as Google, without trusting credentials issued to third parties. Admins who
wish to utilize multiple OAuth clients should explore providers which support the
`azp` (authorized party) claim, a mechanism for allowing one client to issue
tokens on behalf of another. -->
重要的是，API服务器不是OAuth2客户端，而是只能配置为信任单个颁发者。
这允许使用Google等公共提供商，而无需信任发给第三方的凭据。
希望使用多个OAuth客户端的管理员应该探索支持“azp”（授权方）声明的提供程序，这是一种允许一个客户端代表另一个客户端发出令牌的机制。

<!--
Kubernetes does not provide an OpenID Connect Identity Provider.
You can use an existing public OpenID Connect Identity Provider (such as Google, or [others](http://connect2id.com/products/nimbus-oauth-openid-connect-sdk/openid-connect-providers)).
Or, you can run your own Identity Provider, such as CoreOS [dex](https://github.com/coreos/dex), [Keycloak](https://github.com/keycloak/keycloak), CloudFoundry [UAA](https://github.com/cloudfoundry/uaa), or Tremolo Security's [OpenUnison](https://github.com/tremolosecurity/openunison).
-->
Kubernetes不提供OpenID Connect身份提供商。
您可以使用现有的公共OpenID Connect身份提供商（例如Google或[其他](http://connect2id.com/products/nimbus-oauth-openid-connect-sdk/openid-connect-providers)）。
或者，您可以运行自己的身份提供程序，例如CoreOS [dex](https://github.com/coreos/dex)，[Keycloak](https://github.com/keycloak/keycloak)，CloudFoundry [UAA](https://github.com/cloudfoundry/uaa)，或Tremolo Security的[OpenUnison](https://github.com/tremolosecurity/openunison)。

<!-- For an identity provider to work with Kubernetes it must: -->
要使身份提供者与Kubernetes合作，它必须：

<!--
1.  Support [OpenID connect discovery](https://openid.net/specs/openid-connect-discovery-1_0.html); not all do.
2.  Run in TLS with non-obsolete ciphers
3.  Have a CA signed certificate (even if the CA is not a commercial CA or is self signed) 
-->
1. 支持[OpenID连接发现](https://openid.net/specs/openid-connect-discovery-1_0.html),不是全部都这样做
2. 使用非过时密码在TLS中运行
3. 拥有CA签名证书（即使CA不是商业CA或自签名）

<!-- 
A note about requirement #3 above, requiring a CA signed certificate.  If you deploy your own identity provider (as opposed to one of the cloud providers like Google or Microsoft) you MUST have your identity provider's web server certificate signed by a certificate with the `CA` flag set to `TRUE`, even if it is self signed.  This is due to GoLang's TLS client implementation being very strict to the standards around certificate validation.  If you don't have a CA handy, you can use [this script](https://github.com/coreos/dex/blob/1ee5920c54f5926d6468d2607c728b71cfe98092/examples/k8s/gencert.sh) from the CoreOS team to create a simple CA and a signed certificate and key pair.
Or you can use [this similar script](https://raw.githubusercontent.com/TremoloSecurity/openunison-qs-kubernetes/master/src/main/bash/makessl.sh) that generates SHA256 certs with a longer life and larger key size. 
-->
关于上述要求 ＃3 的说明，需要CA签名证书。如果您部署自己的身份提供商（而不是Google或Microsoft等云提供商之一），则必须让您的身份提供商的Web服务器证书由证书签名，并且`CA`标志设置为`TRUE`，即使它是自签名。
这是因为GoLang的TLS客户端实现对证书验证的标准非常严格。
如果您没有CA，可以使用CoreOS团队的[此脚本](https://github.com/coreos/dex/blob/1ee5920c54f5926d6468d2607c728b71cfe98092/examples/k8s/gencert.sh)
创建一个简单的CA和签名证书和密钥对。
或者您可以使用[此类似脚本](https://raw.githubusercontent.com/TremoloSecurity/openunison-qs-kubernetes/master/src/main/bash/makessl.sh)
生成具有更长寿命和更长寿命的SHA256证书密钥大小。

<!-- Setup instructions for specific systems: -->
特定系统的安装说明：

- [UAA](http://apigee.com/about/blog/engineering/kubernetes-authentication-enterprise)
- [Dex](https://speakerdeck.com/ericchiang/kubernetes-access-control-with-dex)
- [OpenUnison](https://github.com/TremoloSecurity/openunison-qs-kubernetes)

<!-- #### Using kubectl -->
#### 使用 kubectl

<!-- ##### Option 1 - OIDC Authenticator -->
##### 选项1  -  OIDC认证

<!--
The first option is to use the kubectl `oidc` authenticator, which sets the `id_token` as a bearer token for all requests and refreshes the token once it expires. After you've logged into your provider, use kubectl to add your `id_token`, `refresh_token`, `client_id`, and `client_secret` to configure the plugin.
-->
第一个选项是使用kubectl`oidc`身份验证器，它将`id_token`设置为所有请求的承载令牌，并在令牌过期后刷新令牌。登录到提供程序后，使用kubectl添加`id_token`，`refresh_token`，`client_id`和`client_secret`来配置插件。

<!--
Providers that don't return an `id_token` as part of their refresh token response (e.g. [Okta](https://developer.okta.com/docs/api/resources/oidc.html#response-parameters-4)) aren't supported by this plugin and should use "Option 2" below.
-->
不作为刷新令牌响应的一部分返回“id_token”的提供者(例如[Okta](https://developer.okta.com/docs/api/resources/oidc.html#response-parameters-4))此插件不支持，应使用下面的“选项2”

```bash
kubectl config set-credentials USER_NAME \
   --auth-provider=oidc \
   --auth-provider-arg=idp-issuer-url=( issuer url ) \
   --auth-provider-arg=client-id=( your client id ) \
   --auth-provider-arg=client-secret=( your client secret ) \
   --auth-provider-arg=refresh-token=( your refresh token ) \
   --auth-provider-arg=idp-certificate-authority=( path to your ca certificate ) \
   --auth-provider-arg=id-token=( your id_token )
```

<!-- As an example, running the below command after authenticating to your identity provider: -->
例如，在向身份提供者进行身份验证后运行以下命令：

```bash
kubectl config set-credentials mmosley  \
        --auth-provider=oidc  \
        --auth-provider-arg=idp-issuer-url=https://oidcidp.tremolo.lan:8443/auth/idp/OidcIdP  \
        --auth-provider-arg=client-id=kubernetes  \
        --auth-provider-arg=client-secret=1db158f6-177d-4d9c-8a8b-d36869918ec5  \
        --auth-provider-arg=refresh-token=q1bKLFOyUiosTfawzA93TzZIDzH2TNa2SMm0zEiPKTUwME6BkEo6Sql5yUWVBSWpKUGphaWpxSVAfekBOZbBhaEW+VlFUeVRGcluyVF5JT4+haZmPsluFoFu5XkpXk5BXqHega4GAXlF+ma+vmYpFcHe5eZR+slBFpZKtQA= \
        --auth-provider-arg=idp-certificate-authority=/root/ca.pem \
        --auth-provider-arg=id-token=eyJraWQiOiJDTj1vaWRjaWRwLnRyZW1vbG8ubGFuLCBPVT1EZW1vLCBPPVRybWVvbG8gU2VjdXJpdHksIEw9QXJsaW5ndG9uLCBTVD1WaXJnaW5pYSwgQz1VUy1DTj1rdWJlLWNhLTEyMDIxNDc5MjEwMzYwNzMyMTUyIiwiYWxnIjoiUlMyNTYifQ.eyJpc3MiOiJodHRwczovL29pZGNpZHAudHJlbW9sby5sYW46ODQ0My9hdXRoL2lkcC9PaWRjSWRQIiwiYXVkIjoia3ViZXJuZXRlcyIsImV4cCI6MTQ4MzU0OTUxMSwianRpIjoiMm96US15TXdFcHV4WDlHZUhQdy1hZyIsImlhdCI6MTQ4MzU0OTQ1MSwibmJmIjoxNDgzNTQ5MzMxLCJzdWIiOiI0YWViMzdiYS1iNjQ1LTQ4ZmQtYWIzMC0xYTAxZWU0MWUyMTgifQ.w6p4J_6qQ1HzTG9nrEOrubxIMb9K5hzcMPxc9IxPx2K4xO9l-oFiUw93daH3m5pluP6K7eOE6txBuRVfEcpJSwlelsOsW8gb8VJcnzMS9EnZpeA0tW_p-mnkFc3VcfyXuhe5R3G7aa5d8uHv70yJ9Y3-UhjiN9EhpMdfPAoEB9fYKKkJRzF7utTTIPGrSaSU6d2pcpfYKaxIwePzEkT4DfcQthoZdy9ucNvvLoi1DIC-UocFD8HLs8LYKEqSxQvOcvnThbObJ9af71EwmuE21fO5KzMW20KtAeget1gnldOosPtz1G5EwvaQ401-RPQzPGMVBld0_zMCAwZttJ4knw
```

<!-- Which would produce the below configuration: -->
会产生以下配置：

```yaml
users:
- name: mmosley
  user:
    auth-provider:
      config:
        client-id: kubernetes
        client-secret: 1db158f6-177d-4d9c-8a8b-d36869918ec5
        id-token: eyJraWQiOiJDTj1vaWRjaWRwLnRyZW1vbG8ubGFuLCBPVT1EZW1vLCBPPVRybWVvbG8gU2VjdXJpdHksIEw9QXJsaW5ndG9uLCBTVD1WaXJnaW5pYSwgQz1VUy1DTj1rdWJlLWNhLTEyMDIxNDc5MjEwMzYwNzMyMTUyIiwiYWxnIjoiUlMyNTYifQ.eyJpc3MiOiJodHRwczovL29pZGNpZHAudHJlbW9sby5sYW46ODQ0My9hdXRoL2lkcC9PaWRjSWRQIiwiYXVkIjoia3ViZXJuZXRlcyIsImV4cCI6MTQ4MzU0OTUxMSwianRpIjoiMm96US15TXdFcHV4WDlHZUhQdy1hZyIsImlhdCI6MTQ4MzU0OTQ1MSwibmJmIjoxNDgzNTQ5MzMxLCJzdWIiOiI0YWViMzdiYS1iNjQ1LTQ4ZmQtYWIzMC0xYTAxZWU0MWUyMTgifQ.w6p4J_6qQ1HzTG9nrEOrubxIMb9K5hzcMPxc9IxPx2K4xO9l-oFiUw93daH3m5pluP6K7eOE6txBuRVfEcpJSwlelsOsW8gb8VJcnzMS9EnZpeA0tW_p-mnkFc3VcfyXuhe5R3G7aa5d8uHv70yJ9Y3-UhjiN9EhpMdfPAoEB9fYKKkJRzF7utTTIPGrSaSU6d2pcpfYKaxIwePzEkT4DfcQthoZdy9ucNvvLoi1DIC-UocFD8HLs8LYKEqSxQvOcvnThbObJ9af71EwmuE21fO5KzMW20KtAeget1gnldOosPtz1G5EwvaQ401-RPQzPGMVBld0_zMCAwZttJ4knw
        idp-certificate-authority: /root/ca.pem
        idp-issuer-url: https://oidcidp.tremolo.lan:8443/auth/idp/OidcIdP
        refresh-token: q1bKLFOyUiosTfawzA93TzZIDzH2TNa2SMm0zEiPKTUwME6BkEo6Sql5yUWVBSWpKUGphaWpxSVAfekBOZbBhaEW+VlFUeVRGcluyVF5JT4+haZmPsluFoFu5XkpXk5BXq
      name: oidc
```
<!-- Once your `id_token` expires, `kubectl` will attempt to refresh your `id_token` using your `refresh_token` and `client_secret` storing the new values for the `refresh_token` and `id_token` in your `.kube/config`. -->
一旦你的`id_token`到期，`kubectl`将尝试使用你的`refresh_token`和`client_secret`刷新你的`id_token`，在`.kube/config`中存储`refresh_token`和`id_token`的新值。


<!-- ##### Option 2 - Use the `--token` Option -->
##### 选项2  - 使用`--token`选项

<!-- The `kubectl` command lets you pass in a token using the `--token` option.  Simply copy and paste the `id_token` into this option: -->
`kubectl`命令允许您使用`--token`选项传入令牌。只需将`id_token`复制并粘贴到此选项中：

```
kubectl --token=eyJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJodHRwczovL21sYi50cmVtb2xvLmxhbjo4MDQzL2F1dGgvaWRwL29pZGMiLCJhdWQiOiJrdWJlcm5ldGVzIiwiZXhwIjoxNDc0NTk2NjY5LCJqdGkiOiI2RDUzNXoxUEpFNjJOR3QxaWVyYm9RIiwiaWF0IjoxNDc0NTk2MzY5LCJuYmYiOjE0NzQ1OTYyNDksInN1YiI6Im13aW5kdSIsInVzZXJfcm9sZSI6WyJ1c2VycyIsIm5ldy1uYW1lc3BhY2Utdmlld2VyIl0sImVtYWlsIjoibXdpbmR1QG5vbW9yZWplZGkuY29tIn0.f2As579n9VNoaKzoF-dOQGmXkFKf1FMyNV0-va_B63jn-_n9LGSCca_6IVMP8pO-Zb4KvRqGyTP0r3HkHxYy5c81AnIh8ijarruczl-TK_yF5akjSTHFZD-0gRzlevBDiH8Q79NAr-ky0P4iIXS8lY9Vnjch5MF74Zx0c3alKJHJUnnpjIACByfF2SCaYzbWFMUNat-K1PaUk5-ujMBG7yYnr95xD-63n8CO8teGUAAEMx6zRjzfhnhbzX-ajwZLGwGUBT4WqjMs70-6a7_8gZmLZb2az1cZynkFRj2BaCkVT3A2RrjeEwZEtGXlMqKJ1_I2ulrOVsYx01_yD35-rw get nodes
```


<!-- ### Webhook Token Authentication -->
### Webhook令牌认证

<!-- Webhook authentication is a hook for verifying bearer tokens. -->
Webhook身份验证是用于验证承载令牌的钩子。

<!--
* `--authentication-token-webhook-config-file` a kubeconfig file describing how to access the remote webhook service.
* `--authentication-token-webhook-cache-ttl` how long to cache authentication decisions. Defaults to two minutes.
-->
* `--authentication-token-webhook-config-file` 一个kubeconfig文件，描述如何访问远程webhook服务。
* `--authentication-token-webhook-cache-ttl` 缓存身份验证决策的时间。默认为两分钟。

<!--
The configuration file uses the [kubeconfig](/docs/concepts/cluster-administration/authenticate-across-clusters-kubeconfig/)
file format. Within the file `users` refers to the API server webhook and
`clusters` refers to the remote service. An example would be:
-->
配置文件使用[kubeconfig](/docs/concepts/cluster-administration/authenticate-across-clusters-kubeconfig/)文件格式。在文件中，`users`指的是API服务器webhook，`clusters`指的是远程服务。一个例子是：

```yaml
# clusters refers to the remote service.
clusters:
  - name: name-of-remote-authn-service
    cluster:
      certificate-authority: /path/to/ca.pem         # CA for verifying the remote service.
      server: https://authn.example.com/authenticate # URL of remote service to query. Must use 'https'.

# users refers to the API server's webhook configuration.
users:
  - name: name-of-api-server
    user:
      client-certificate: /path/to/cert.pem # cert for the webhook plugin to use
      client-key: /path/to/key.pem          # key matching the cert

# kubeconfig files require a context. Provide one for the API server.
current-context: webhook
contexts:
- context:
    cluster: name-of-remote-authn-service
    user: name-of-api-sever
  name: webhook
```

<!--
When a client attempts to authenticate with the API server using a bearer token
as discussed [above](#putting-a-bearer-token-in-a-request),
the authentication webhook POSTs a JSON-serialized `authentication.k8s.io/v1beta1` `TokenReview` object containing the token
to the remote service. Kubernetes will not challenge a request that lacks such a header.
-->
当客户端尝试使用承载令牌向API服务器进行身份验证时，如上所述（#put-a-bearer-token-in-a-request），
身份验证webhook将包含令牌的JSON序列化`authentication.k8s.io/v1beta1` `TokenReview`对象POST到远程服务。 Kubernetes不会挑战缺少这种标题的请求。

<!--
Note that webhook API objects are subject to the same [versioning compatibility rules](/docs/concepts/overview/kubernetes-api/)
as other Kubernetes API objects. Implementers should be aware of looser
compatibility promises for beta objects and check the "apiVersion" field of the
request to ensure correct deserialization. Additionally, the API server must
enable the `authentication.k8s.io/v1beta1` API extensions group (`--runtime-config=authentication.k8s.io/v1beta1=true`).
-->
请注意，webhook API对象与其他Kubernetes API对象具有相同的[版本兼容性规则](/docs/concepts/overview/kubernetes-api/)。实施者应该了解beta对象的更松散的兼容性承诺，并检查请求的“apiVersion”字段以确保正确的反序列化。此外，API服务器必须启用`authentication.k8s.io/v1beta1` API扩展组（`--runtime-config=authentication.k8s.io/v1beta1=true`）。

<!-- The POST body will be of the following format: -->
POST正文将采用以下格式：

```json
{
  "apiVersion": "authentication.k8s.io/v1beta1",
  "kind": "TokenReview",
  "spec": {
    "token": "(BEARERTOKEN)"
  }
}
```

<!--
The remote service is expected to fill the `status` field of
the request to indicate the success of the login. The response body's `spec`
field is ignored and may be omitted. A successful validation of the bearer
token would return:
-->
远程服务应填充请求的“status”字段以指示登录成功。响应主体的`spec`字段被忽略，可以省略。成功验证承载令牌将返回：

```json
{
  "apiVersion": "authentication.k8s.io/v1beta1",
  "kind": "TokenReview",
  "status": {
    "authenticated": true,
    "user": {
      "username": "janedoe@example.com",
      "uid": "42",
      "groups": [
        "developers",
        "qa"
      ],
      "extra": {
        "extrafield1": [
          "extravalue1",
          "extravalue2"
        ]
      }
    }
  }
}
```

<!-- An unsuccessful request would return: -->
不成功的请求将返回：

```json
{
  "apiVersion": "authentication.k8s.io/v1beta1",
  "kind": "TokenReview",
  "status": {
    "authenticated": false
  }
}
```

<!-- HTTP status codes can be used to supply additional error context. -->
HTTP状态代码可用于提供其他错误上下文。


<!-- ### Authenticating Proxy -->
### Authenticating Proxy

<!--
The API server can be configured to identify users from request header values, such as `X-Remote-User`.
It is designed for use in combination with an authenticating proxy, which sets the request header value.
-->
API服务器可以配置为从请求标头值中识别用户，例如“X-Remote-User”。
它旨在与身份验证代理结合使用，该代理设置请求标头值。

<!--
* `--requestheader-username-headers` Required, case-insensitive. Header names to check, in order, for the user identity. The first header containing a value is used as the username.
* `--requestheader-group-headers` 1.6+. Optional, case-insensitive. "X-Remote-Group" is suggested. Header names to check, in order, for the user's groups. All values in all specified headers are used as group names.
* `--requestheader-extra-headers-prefix` 1.6+. Optional, case-insensitive. "X-Remote-Extra-" is suggested. Header prefixes to look for to determine extra information about the user (typically used by the configured authorization plugin). Any headers beginning with any of the specified prefixes have the prefix removed, the remainder of the header name becomes the extra key, and the header value is the extra value.
-->
* `--requestheader-username-headers` 必需，不区分大小写。用于标识用户标识的标题名称。包含值的第一个标头用作用户名。
* `--requestheader-group-headers` 1.6+. 可选，不区分大小写。建议使用“X-Remote-Group”。要按顺序检查用户组的标题名称。所有指定标头中的所有值都用作组名
* `--requestheader-extra-headers-prefix` 1.6+. 可选，不区分大小写。建议使用“X-Remote-Extra-”。标头前缀用于查找以确定有关用户的额外信息（通常由配置的授权插件使用）。任何以任何指定前缀开头的标头都会删除前缀，标题名称的其余部分将成为额外键，标头值将是额外值。

<!-- For example, with this configuration: -->
例如，使用此配置：

```
--requestheader-username-headers=X-Remote-User
--requestheader-group-headers=X-Remote-Group
--requestheader-extra-headers-prefix=X-Remote-Extra-
```

<!-- this request: -->
这个请求：

```http
GET / HTTP/1.1
X-Remote-User: fido
X-Remote-Group: dogs
X-Remote-Group: dachshunds
X-Remote-Extra-Scopes: openid
X-Remote-Extra-Scopes: profile
```

<!-- would result in this user info: -->
会导致这样的用户信息：

```yaml
name: fido
groups:
- dogs
- dachshunds
extra:
  scopes:
  - openid
  - profile
```

<!--
In order to prevent header spoofing, the authenticating proxy is required to present a valid client
certificate to the API server for validation against the specified CA before the request headers are
checked.
-->
为了防止标头欺骗，需要验证代理提供有效的客户端证书到API服务器，以便在请求标头之前针对指定的CA进行验证检查。

<!--
* `--requestheader-client-ca-file` Required. PEM-encoded certificate bundle. A valid client certificate must be presented and validated against the certificate authorities in the specified file before the request headers are checked for user names.
* `--requestheader-allowed-names` Optional.  List of common names (cn). If set, a valid client certificate with a Common Name (cn) in the specified list must be presented before the request headers are checked for user names. If empty, any Common Name is allowed.
-->
* `--requestheader-client-ca-file` 需要。 PEM编码的证书包。在检查用户名的请求标头之前，必须针对指定文件中的证书颁发机构提供和验证有效的客户端证书。
* `--requestheader-allowed-names` 可选的。常用名称列表（cn）。如果设置，则在检查用户名的请求标头之前，必须显示指定列表中具有公用名（cn）的有效客户端证书。如果为空，则允许任何公共名称。


<!-- ## Anonymous requests -->
## 匿名请求

<!--
When enabled, requests that are not rejected by other configured authentication methods are
treated as anonymous requests, and given a username of `system:anonymous` and a group of
`system:unauthenticated`.
-->
启用后，未被其他已配置的身份验证方法拒绝的请求将被视为匿名请求，并提供用户名“system：anonymous”和一组`system：unauthenticated`。

<!--
For example, on a server with token authentication configured, and anonymous access enabled,
a request providing an invalid bearer token would receive a `401 Unauthorized` error.
A request providing no bearer token would be treated as an anonymous request.
-->
例如，在配置了令牌认证并启用匿名访问的服务器上，提供无效承载令牌的请求将收到“401 Unauthorized”错误。不提供承载令牌的请求将被视为匿名请求。

<!--
In 1.5.1-1.5.x, anonymous access is disabled by default, and can be enabled by
passing the `--anonymous-auth=true` option to the API server.
-->
在1.5.1-1.5.x中，默认情况下禁用匿名访问，可以通过将`--anonymous-auth=true`选项传递给API服务器来启用。

<!--
In 1.6+, anonymous access is enabled by default if an authorization mode other than `AlwaysAllow`
is used, and can be disabled by passing the `--anonymous-auth=false` option to the API server.
Starting in 1.6, the ABAC and RBAC authorizers require explicit authorization of the
`system:anonymous` user or the `system:unauthenticated` group, so legacy policy rules
that grant access to the `*` user or `*` group do not include anonymous users.
-->
在1.6+中，如果使用除“AlwaysAllow”之外的授权模式，则默认启用匿名访问，并且可以通过将`--anonymous-auth=false`选项传递给API服务器来禁用匿名访问。从1.6开始，ABAC和RBAC授权者需要明确授权`system:anonymous`用户或`system:unauthenticated`组，因此授予访问`*`用户或`*`组的旧策略规则不包括匿名用户。

<!-- ## User impersonation -->
## 模仿用户

<!--
A user can act as another user through impersonation headers. These let requests
manually override the user info a request authenticates as. For example, an admin
could use this feature to debug an authorization policy by temporarily
impersonating another user and seeing if a request was denied.
-->
用户可以通过模拟标头充当另一个用户。这些let请求会手动覆盖请求进行身份验证的用户信息。例如，管理员可以使用此功能通过临时模拟其他用户并查看请求是否被拒绝来调试授权策略。

<!--
Impersonation requests first authenticate as the requesting user, then switch
to the impersonated user info.
-->
模拟请求首先作为请求用户进行身份验证，然后切换到模拟的用户信息。

<!--
* A user makes an API call with their credentials _and_ impersonation headers.
* API server authenticates the user.
* API server ensures the authenticated users have impersonation privileges.
* Request user info is replaced with impersonation values.
* Request is evaluated, authorization acts on impersonated user info.
-->
* 用户使用其凭据_and_模拟标头进行API调用。
* API服务器对用户进行身份验证。
* API服务器确保经过身份验证的用户具有模拟权限。
* 请求用户信息被替换为模拟值。
* 评估请求，授权对模拟的用户信息起作用。

<!-- The following HTTP headers can be used to performing an impersonation request: -->
以下HTTP标头可用于执行模拟请求：

<!--
* `Impersonate-User`: The username to act as.
* `Impersonate-Group`: A group name to act as. Can be provided multiple times to set multiple groups. Optional. Requires "Impersonate-User"
* `Impersonate-Extra-( extra name )`: A dynamic header used to associate extra fields with the user. Optional. Requires "Impersonate-User"
-->
*`Impersonate-User`：用作的用户名。
*`Impersonate-Group`：一个组名称。可以多次提供以设置多个组。可选的。需要“模仿用户”
*`Impersonate-Extra-（额外名称）`：用于将额外字段与用户关联的动态标头。可选的。需要“模仿用户”

<!-- An example set of headers: -->
An example set of headers:

```http
Impersonate-User: jane.doe@example.com
Impersonate-Group: developers
Impersonate-Group: admins
Impersonate-Extra-dn: cn=jane,ou=engineers,dc=example,dc=com
Impersonate-Extra-scopes: view
Impersonate-Extra-scopes: development
```

<!--
When using `kubectl` set the `--as` flag to configure the `Impersonate-User`
header, set the `--as-group` flag to configure the `Impersonate-Group` header.
-->
当使用`kubectl`时，设置`--as`标志来配置`Impersonate-User`标头，设置`--as-group`标志以配置`Impersonate-Group`标头。

```shell
$ kubectl drain mynode
Error from server (Forbidden): User "clark" cannot get nodes at the cluster scope. (get nodes mynode)

$ kubectl drain mynode --as=superman --as-group=system:masters
node "mynode" cordoned
node "mynode" drained
```

<!--
To impersonate a user, group, or set extra fields, the impersonating user must
have the ability to perform the "impersonate" verb on the kind of attribute
being impersonated ("user", "group", etc.). For clusters that enable the RBAC
authorization plugin, the following ClusterRole encompasses the rules needed to
set user and group impersonation headers:
-->
要模拟用户，分组或设置额外字段，模仿用户必须能够对被模拟的属性（“用户”，“组”等）执行“模拟”动词。对于启用RBAC授权插件的集群，以下ClusterRole包含设置用户和组模拟标头所需的规则：

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: impersonator
rules:
- apiGroups: [""]
  resources: ["users", "groups", "serviceaccounts"]
  verbs: ["impersonate"]
```

<!--
Extra fields are evaluated as sub-resources of the resource "userextras". To
allow a user to use impersonation headers for the extra field "scopes," a user
should be granted the following role:
-->
额外字段被评估为资源“userextras”的子资源。要允许用户对额外字段“范围”使用模拟标头，应授予用户以下角色：

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: scopes-impersonator
rules:
# Can set "Impersonate-Extra-scopes" header.
- apiGroups: ["authentication.k8s.io"]
  resources: ["userextras/scopes"]
  verbs: ["impersonate"]
```

<!--
The values of impersonation headers can also be restricted by limiting the set
of `resourceNames` a resource can take.
-->
还可以通过限制资源可以采用的“resourceNames”集来限制模拟头的值。

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: limited-impersonator
rules:
# Can impersonate the user "jane.doe@example.com"
- apiGroups: [""]
  resources: ["users"]
  verbs: ["impersonate"]
  resourceNames: ["jane.doe@example.com"]

# Can impersonate the groups "developers" and "admins"
- apiGroups: [""]
  resources: ["groups"]
  verbs: ["impersonate"]
  resourceNames: ["developers","admins"]

# Can impersonate the extras field "scopes" with the values "view" and "development"
- apiGroups: ["authentication.k8s.io"]
  resources: ["userextras/scopes"]
  verbs: ["impersonate"]
  resourceNames: ["view", "development"]
```

<!-- ## client-go credential plugins -->
## client-go凭证插件

{{< feature-state for_k8s_version="v1.11" state="beta" >}}

<!--
`k8s.io/client-go` and tools using it such as `kubectl` and `kubelet` are able to execute an
external command to receive user credentials.
-->
`k8s.io/client-go`和使用它的工具如`kubectl`和`kubelet`能够执行外部命令来接收用户凭证。

<!--
This feature is intended for client side integrations with authentication protocols not natively
supported by `k8s.io/client-go` (LDAP, Kerberos, OAuth2, SAML, etc.). The plugin implements the
protocol specific logic, then returns opaque credentials to use. Almost all credential plugin
use cases require a server side component with support for the [webhook token authenticator](#webhook-token-authentication)
to interpret the credential format produced by the client plugin.
-->
此功能旨在与客户端集成，使用“k8s.io/client-go”（LDAP，Kerberos，OAuth2，SAML等）本身不支持的身份验证协议。该插件实现协议特定的逻辑，然后返回不透明的凭据以供使用。几乎所有凭证插件用例都需要服务器端组件支持[webhook令牌验证器](#Webhook令牌认证)来解释客户端插件生成的凭据格式。

<!-- ### Example use case -->
### 示例用例

<!--
In a hypothetical use case, an organization would run an external service that exchanges LDAP credentials
for user specific, signed tokens. The service would also be capable of responding to [webhook token
authenticator](#webhook-token-authentication) requests to validate the tokens. Users would be required
to install a credential plugin on their workstation.
-->
在假设的用例中，组织将运行外部服务，该服务为特定于用户的签名令牌交换LDAP凭据。该服务还能够响应[webhook令牌验证器](#Webhook令牌认证)请求来验证令牌。用户需要在他们的工作站上安装凭证插件。

<!-- To authenticate against the API: -->
要对API进行身份验证：

<!--
* The user issues a `kubectl` command.
* Credential plugin prompts the user for LDAP credentials, exchanges credentials with external service for a token.
* Credential plugin returns token to client-go, which uses it as a bearer token against the API server.
* API server uses the [webhook token authenticator](#webhook-token-authentication) to submit a `TokenReview` to the external service.
* External service verifies the signature on the token and returns the user's username and groups.
-->
* 用户发出`kubectl`命令。
* Credential插件会提示用户输入LDAP凭据，与令牌的外部服务交换凭据。
* Credential插件将令牌返回给client-go，后者将其用作针对API服务器的承载令牌。
* API服务器使用[webhook令牌身份验证器]（#webhook-token-authentication）向外部服务提交`TokenReview`。
* 外部服务验证令牌上的签名并返回用户的用户名和组。

<!-- ### Configuration -->
### 配置

<!--
Credential plugins are configured through [kubectl config files](/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
as part of the user fields.
-->
凭据插件通过[kubectl配置文件](/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)配置为用户字段的一部分。

```yaml
apiVersion: v1
kind: Config
users:
- name: my-user
  user:
    exec:
      # Command to execute. Required.
      command: "example-client-go-exec-plugin"

      # API version to use when decoding the ExecCredentials resource. Required.
      #
      # The API version returned by the plugin MUST match the version listed here.
      #
      # To integrate with tools that support multiple versions (such as client.authentication.k8s.io/v1alpha1),
      # set an environment variable or pass an argument to the tool that indicates which version the exec plugin expects.
      apiVersion: "client.authentication.k8s.io/v1beta1"

      # Environment variables to set when executing the plugin. Optional.
      env:
      - name: "FOO"
        value: "bar"

      # Arguments to pass when executing the plugin. Optional.
      args:
      - "arg1"
      - "arg2"
clusters:
- name: my-cluster
  cluster:
    server: "https://172.17.4.100:6443"
    certificate-authority: "/etc/kubernetes/ca.pem"
contexts:
- name: my-cluster
  context:
    cluster: my-cluster
    user: my-user
current-context: my-cluster
```

<!--
Relative command paths are interpreted as relative to the directory of the config file. If
KUBECONFIG is set to `/home/jane/kubeconfig` and the exec command is `./bin/example-client-go-exec-plugin`,
the binary `/home/jane/bin/example-client-go-exec-plugin` is executed.
-->
相对命令路径被解释为相对于配置文件的目录。如果KUBECONFIG设置为`/home/jane/kubeconfig`并且exec命令是`./bin/example-client-go-exec-plugin`，那么二进制文件`/home/jane/bin/example-client-go-exec-plugin`被执行。

```yaml
- name: my-user
  user:
    exec:
      # Path relative to the directory of the kubeconfig
      command: "./bin/example-client-go-exec-plugin"
      apiVersion: "client.authentication.k8s.io/v1beta1"
```

<!-- ### Input and output formats -->
### 输入和输出格式

<!--
The executed command prints an `ExecCredential` object to `stdout`. `k8s.io/client-go`
authenticates against the Kubernetes API using the returned credentials in the `status`.
-->
执行的命令将`ExecCredential`对象打印到`stdout`。 `k8s.io/client-go`使用`status`中返回的凭据对Kubernetes API进行身份验证。

<!--
When run from an interactive session, `stdin` is exposed directly to the plugin. Plugins should use a
[TTY check](https://godoc.org/golang.org/x/crypto/ssh/terminal#IsTerminal) to determine if it's
appropriate to prompt a user interactively.
-->
从交互式会话运行时，`stdin`直接暴露给插件。插件应使用[TTY检查](https://godoc.org/golang.org/x/crypto/ssh/terminal#IsTerminal)来确定是否适合以交互方式提示用户。

<!--
To use bearer token credentials, the plugin returns a token in the status of the `ExecCredential`.
-->
要使用承载令牌凭证，插件将返回“ExecCredential”状态的令牌。

```json
{
  "apiVersion": "client.authentication.k8s.io/v1beta1",
  "kind": "ExecCredential",
  "status": {
    "token": "my-bearer-token"
  }
}
```

<!--
Alternatively, a PEM-encoded client certificate and key can be returned to use TLS client auth.
If the plugin returns a different certificate and key on a subsequent call, `k8s.io/client-go` 
will close existing connections with the server to force a new TLS handshake.
-->
或者，可以返回PEM编码的客户端证书和密钥以使用TLS客户端身份验证。
如果插件在后续调用中返回不同的证书和密钥，`k8s.io/client-go`将关闭与服务器的现有连接以强制进行新的TLS握手。

<!-- If specified, `clientKeyData` and `clientCertificateData` must both must be present. -->
如果指定，`clientKeyData`和`clientCertificateData`必须都必须存在。

<!-- `clientCertificateData` may contain additional intermediate certificates to send to the server. -->
`clientCertificateData`可能包含要发送到服务器的其他中间证书。

```json
{
  "apiVersion": "client.authentication.k8s.io/v1beta1",
  "kind": "ExecCredential",
  "status": {
    "clientCertificateData": "-----BEGIN CERTIFICATE-----\n...\n-----END CERTIFICATE-----",
    "clientKeyData": "-----BEGIN RSA PRIVATE KEY-----\n...\n-----END RSA PRIVATE KEY-----"
  }
}
```

<!--
Optionally, the response can include the expiry of the credential formatted as a
RFC3339 timestamp. Presence or absence of an expiry has the following impact:
-->
可选地，响应可以包括格式化为RFC3339时间戳的凭证到期。到期日的存在与否会产生以下影响：

<!--
- If an expiry is included, the bearer token and TLS credentials are cached until
  the expiry time is reached, or if the server responds with a 401 HTTP status code,
  or when the process exits.
- If an expiry is omitted, the bearer token and TLS credentials are cached until
  the server responds with a 401 HTTP status code or until the process exits.
-->
 - 如果包含到期日，则承载令牌和TLS凭证将被缓存，直到达到到期时间，或者服务器以401 HTTP状态代码响应，或者进程退出时。
 - 如果省略到期，则缓存承载令牌和TLS凭证，直到服务器以401 HTTP状态代码响应或直到进程退出。

```json
{
  "apiVersion": "client.authentication.k8s.io/v1beta1",
  "kind": "ExecCredential",
  "status": {
    "token": "my-bearer-token",
    "expirationTimestamp": "2018-03-05T17:30:20-08:00"
  }
}
```
{{% /capture %}}
