---
title: Installing Rancher Server with SSL
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---

## 安装Rancher并使用SSL

---

为了在Rancher server启用 `https` 访问，你需要在Rancher server前使用一个代理服务器代理https请求，并能设置http的头参数。我们会在以下的内容中提供一个使用NGINX、HAProxy或者Apache作为代理的例子。当然了，其他工具也是可以的。

### 需求

出了一般的Rancher server[需求]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/installing-rancher/installing-server/#requirements)外，你还需要：

* 有效的SSL证书：如果你的证书并不是标准的Ubuntu CA bundle，请参考以下内容[self signed certificate instructions]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/installing-rancher/installing-server/basic-ssl-config/#using-self-signed-certs-beta)。
* 相关域名的DNS配置

### Rancher Server 标签

Rancher server当前版本中有2个不同的标签。对于每一个主要的release标签，我们都会提供对应版本的文档。

* `rancher/server:latest` 此标签是我们的最新一次开发的构建版本。这些构建已经被我们的CI框架自动验证测试。但这些release并不代表可以在生产环境部署。
* `rancher/server:stable` 此标签是我们最新一个稳定的release构建。这个标签代表我们推荐在生产环境中使用的版本。

请不要使用任何带有 `rc{n}` 前缀的release。这些构建都是Rancher团队的测试构建。

### 启动 Rancher Server

In our example configuration, all traffic will pass through the proxy and be sent over a Docker link to the Rancher server container. There are alternative approaches that could be followed, but this example is simple and translates well.
在我们的例子配置中，所有的流量都会通过一个Docker link从代理传入Rancher server容器。有其他替代的方法，但在我们的例子中会尽量的简单易懂

启动Rancher server。我们需要添加 `--name=rancher-server` 参数到命令中，使得代理的容器可以与Rancher server容器建立Docker link

```bash
$ sudo docker run -d --restart=unless-stopped --name=rancher-server rancher/server
```
<br>

> **Note:** 在我们的例子中，我们假设代理会运行在其他容器中。如果你打算在其他的服务器上运行代理，则你需要在Rancher server上暴露8080端口，本地的话，通过在 `docker run` 中添加 `-p 127.0.0.1:8080:8080` 参数。

如果你需要复用现有的Rancher server实例，升级的步骤会根据你如何运行原有的Rancher实例而不同。

* 使用容器内部MySQL数据库的Rancher实例，可以参考[upgrade instructions]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/upgrading/#upgrading-rancher-by-creating-a-data-container) 去创建一个data_container，并在新运行的Rancher server实例的命令时，加入 `--volumes-from=<data_container>` 参数。
* 使用 [bind mounted database]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/installing-rancher/installing-server/#single-container-bind-mount) 的实例，请参考 [upgrade instructions for bind mounted instances]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/upgrading/#single-container-bind-mount)。
* 对于使用外部数据库的Rancher实例，停止并移除现有的Rancher容器，新建一个容器即可 [instructions for connecting to an external database]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/installing-rancher/installing-server/#single-container-external-database)。

### Nginx 配置模版

以下是最小的NGINX配置。你应该根据你的需要定制化你自己的配置。本配置需要nginx版本大于 1.9.5。

#### 设置注意项

* `rancher-server` 是你的Rancher server容器的名称。 当你启动Rancher server容器时，命令中必须包括 `--name=rancher-server` 参数。当你启动nginx容器时，你的命令则必须包括 `--link=rancher-server` ，这样以下的配置才能生效。
* `<server>` 可以是任何的名字，但是必须与http/https server配置的名称一致。


```
upstream rancher {
    server rancher-server:8080;
}

map $http_upgrade $connection_upgrade {
    default Upgrade;
    ''      close;
}

server {
    listen 443 ssl spdy;
    server_name <server>;
    ssl_certificate <cert_file>;
    ssl_certificate_key <key_file>;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port $server_port;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://rancher;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        # This allows the ability for the execute shell window to remain open for up to 15 minutes. Without this parameter, the default is 1 minute and will automatically close.
        proxy_read_timeout 900s;
    }
}

server {
    listen 80;
    server_name <server>;
    return 301 https://$server_name$request_uri;
}
```


### Apache 配置例子


以下是使用Apache作为负载均衡的配置例子。

#### 设置注意项

* `<server_name>` 是Rancher server容器的名称。当你启动Apache容器，命令中必须包含 `--link=<server_name>`，这样以下的配置才能生效。
* 在代理的设置中，你需要在配置中替换 `rancher` 这个参数。
* 确保 `proxy_wstunnel` 这个参数是启用的（websocket支持）。

```
<VirtualHost *:80>
  ServerName <server_name>
  Redirect / https://<server_name>/
</VirtualHost>

<VirtualHost *:443>
  ServerName <server_name>

  SSLEngine on
  SSLCertificateFile </path/to/ssl/cert_file>
  SSLCertificateKeyFile </path/to/ssl/key_file>

  ProxyRequests Off
  ProxyPreserveHost On

  RewriteEngine On
  RewriteCond %{HTTP:Connection} Upgrade [NC]
  RewriteCond %{HTTP:Upgrade} websocket [NC]
  RewriteRule /(.*) ws://rancher:8080/$1 [P,L]

  RequestHeader set X-Forwarded-Proto "https"
  RequestHeader set X-Forwarded-Port "443"

  <Location />
    ProxyPass "http://rancher:8080/"
    ProxyPassReverse "http://rancher:8080/"
  </Location>

</VirtualHost>
```

### HAProxy 配置例子

以下是HAProxy的最小配置。你应该根据你的需要去修改。

#### 设置注意项

* `<rancher_server_X_IP>`是Rancher servers的IP地址。


```
global
  maxconn 4096
  ssl-server-verify none

defaults
  mode http
  balance roundrobin
  option redispatch
  option forwardfor

  timeout connect 5s
  timeout queue 5s
  timeout client 36000s
  timeout server 36000s

frontend http-in
  mode http
  bind *:443 ssl crt /etc/haproxy/certificate.pem
  default_backend rancher_servers

  # Add headers for SSL offloading
  http-request set-header X-Forwarded-Proto https if { ssl_fc }
  http-request set-header X-Forwarded-Ssl on if { ssl_fc }

  acl is_websocket hdr(Upgrade) -i WebSocket
  acl is_websocket hdr_beg(Host) -i ws
  use_backend rancher_servers if is_websocket

backend rancher_servers
  server websrv1 <rancher_server_1_IP>:8080 weight 1 maxconn 1024
  server websrv2 <rancher_server_2_IP>:8080 weight 1 maxconn 1024
  server websrv3 <rancher_server_3_IP>:8080 weight 1 maxconn 1024
```

### 更新Host注册信息

使用以上的配置运行Rancher后，UI访问地址变成了`https://<your domain>/`。

在[adding hosts]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/)之前，你需要适当地为SSL配置[Host Registration]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/settings/#host-registration)。

<a id="elb"></a>

### 使用AWS的Elastic Load Balancer作为Rancher Server HA的负载均衡器并使用SSL

我们建议使用AWS的ELB作为你Rancher server的负载均衡器。为了让ELB与Rancher的websockets正常工作，你需要开启proxy protocol模式并且保证HTTP support被停用。 默认的，ELB是在HTTP/HTTPS模式启用，在这个模式下不支持websockets。listener的配置需要被特别的关注。

#### Listener 配置 - SSL

在ELB的SSL控制台，listener配置与以下配置类似：

| Configuration Type | Load Balancer Protocol | Load Balancer Port | Instance Protocol | Instance Port |
|---|---|---|---|---|
| SSL-Terminated | SSL (Secure TCP) | 443 | TCP | 8080 (或者使用启动Rancher时配置 `--advertise-http-port` 的端口) |

* 需要添加相应的安全组设置以及SSL证书

#### 启用 Proxy Protocol

为了使websockets正常工作，ELB的proxy protocol policy必须被启用。

* 启用 [proxy protocol](http://docs.aws.amazon.com/ElasticLoadBalancing/latest/DeveloperGuide/enable-proxy-protocol.html) 模式

```
$ aws elb create-load-balancer-policy --load-balancer-name <LB_NAME> --policy-name <POLICY_NAME> --policy-type-name ProxyProtocolPolicyType --policy-attributes AttributeName=ProxyProtocol,AttributeValue=true
$ aws elb set-load-balancer-policies-for-backend-server --load-balancer-name <LB_NAME> --instance-port 443 --policy-names <POLICY_NAME>
$ aws elb set-load-balancer-policies-for-backend-server --load-balancer-name <LB_NAME> --instance-port 8080 --policy-names <POLICY_NAME>
```

* Health check可以配置使用HTTP:8080下的 `/ping` 路径进行健康检查

<a id="alb"></a>

### 使用AWS的Application Load Balancer(ALB) 作为Rancher Server HA的负载均衡器

我们不再推荐使用AWS的Application Load Balancer (ALB)替代Elastic/Classic Load Balancer (ELB)。如果你依然选择使用ALB，你需要直接指定流量到Rancher server节点上的HTTP端口，默认是8080。

> **Note:** If you use an ALB with Kuberenetes, `kubectl exec` will not work and for that functionality, you will need to use an ELB.
> **Note:** 如果你使用ALB配合Kubernetes，`kubectl exec` 并不能使用那个功能，你需要使用ELB。

### 使用自签名证书 (Beta)

#### 弃用

以下的配置只会在Rancher的核心服务并且是单节点部署模式下生效。当前并没有官方的Rancher模版[Rancher catalog](https://github.com/rancher/rancher-catalog)支持。

Rancher Compose CLI 将需要CA证书，作为操作系统中部分默认配置。请参考[Golang root_*](https://golang.org/src/crypto/x509/)。

#### 前置条件

* PEM格式的CA 证书
* 为Rancher Server签名的CA证书
* Nginx或者Apache实例，反向代理Rancher Server，并配置SSL

#### Rancher Server

1. 通过以下的Docker命令启动Rancher server容器。证书**必须**放在容器内部`/var/lib/rancher/etc/ssl/ca.crt`的位置。

   ```bash
   $ sudo docker run -d --restart=unless-stopped -p 8080:8080 -v /some/dir/cert.crt:/var/lib/rancher/etc/ssl/ca.crt rancher/server
   ```
    <br>

    > **Note:** If you are running NGINX or Apache in a container, you can directly link the instance and not publish the Rancher UI 8080 port.

    The command will configure the server's ca-certificate bundle so that the Rancher services for machine provisioning, catalog and compose executor can communicate with the Rancher server.

2. 如果你使用Nginx或者Apache代理SSL，在容器启动命令中添加`--link=<rancher_server_container_name>`参数。

3. 使用 `https` 地址访问Rancher，例如 `https://rancher.server.domain`。

4. 为SSL更新 [Host Registration]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/settings/#host-registration)配置

> **Note:** Unless the machine running your web browser trusts the CA certificate used to sign the Rancher server certificate, the browser will give an untrusted site warning whenever you visit the web page.

#### 添加 Hosts

1. 在你准遍添加到Rancher集群的Host上，使用PEM格式保存CA证书，并放入 `/var/lib/rancher/etc/ssl` 文件夹下并改名为 `ca.crt`。

2. Add the [custom host]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/custom/), which is just copying and pasting the command from the UI. The command will already include  `-v /var/lib/rancher:/var/lib/rancher`, so the file will automatically be copied onto your host.
2. 添加 [custom host]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/custom/), 使用UI上提示的命令复制到Host上. 命令回自动挂载目录 `-v /var/lib/rancher:/var/lib/rancher`, 所以文件回自动的复制到你的Host上。
