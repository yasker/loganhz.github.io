---
title: Installing Rancher Server
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
redirect_from:
  - /rancher/installing-rancher/installing-server/
  - /rancher/latest/en/installing-rancher/installing-server/
---

## 安装 Rancher Server
---
Rancher是使用一系列的Docker容器进行部署的。运行Rancher跟启动两个容器一样简单。一个容器作为管理服务器部署，另外一个作为集群节点的Agent部署

* [Rancher Server - 单容器部署 (non-HA)](#single-container)
* [Rancher Server - 单容器部署 (non-HA) - 使用外置数据库](#single-container-external-database)
* [Rancher Server - 单容器部署 (non-HA)- 挂载MySQL数据库的数据目录](#single-container-bind-mount)
* [Rancher Server - 多节点的HA部署](#multi-nodes)
* [Rancher Server - 使用AWS的Elastic/Classic Load Balancer作为Rancher Server HA的负载均衡器](#elb)
* [Rancher Server - TLS认证使用AD/OpenLDAP](#ldap)
* [Rancher Server - 在HTTP代理后方启动 Rancher Server](#http-proxy)


> **Note:** 你可以运行Rancher server的容器的命令 `docker run rancher/server --help` 来获得所有选项以及帮助信息。

### 安装需求

* 所有现在的Linux分发版并使用 [支持的Docker版本]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/#supported-docker-versions)。 [RancherOS](http://docs.rancher.com/os/), Ubuntu, RHEL/CentOS 7 都是经过严格的测试。
  * 对于 RHEL/CentOS, 默认的 storage driver, 例如 devicemapper using loopback, 并不被[Docker](https://docs.docker.com/engine/reference/commandline/dockerd/#/storage-driver-options)推荐。 请参考Docker的文档去修改使用其他的storage driver。
  * 对于 RHEL/CentOS, 如果你想使用 SELinux, 你需要 [安装额外的 SELinux 组件]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/installing-rancher/selinux/).
* 1GB RAM
* MySQL server 需要 max_connections 的设置 > 150
  * MYSQL 配置需求
    * Option 1: Run with Antelope with default of `COMPACT`
    * Option 2: Run MySQL 5.7 with Barracuda where the default `ROW_FORMAT` is `Dynamic`

> **Note:** 目前Rancher中并不支持Docker for Mac

### Rancher Server 标签

Rancher server当前版本中有2个不同的标签。对于每一个主要的release标签，我们都会提供对应版本的文档。

* `rancher/server:latest` 此标签是我们的最新一次开发的构建版本。这些构建已经被我们的CI框架自动验证测试。但这些release并不代表可以在生产环境部署。
* `rancher/server:stable` 此标签是我们最新一个稳定的release构建。这个标签代表我们推荐在生产环境中使用的版本。

请不要使用任何带有 `rc{n}` 前缀的release。这些构建都是Rancher团队的测试构建。

<a id="single-container"></a>

### 启动 Rancher Server - 单容器部署 (non-HA)

在安装了Docker的Linux服务器上，使用一个简单的命令就可以启动一个单实例的Rancher。

```bash
$ sudo docker run -d --restart=unless-stopped -p 8080:8080 rancher/server
```

### Rancher UI

UI以及API会使用 `8080` 端口对外服务。下载Docker镜像完成后，需要1到2分钟的时间Rancher才能完全启动并提供服务。

访问如下的URL: `http://<SERVER_IP>:8080`。`<SERVER_IP>` 是运行Rancher server的host的公共IP地址。

当UI已经启动并运行，你可以先进行[adding hosts]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/) 或者在 Infrastructure catalog 中选择一个容器编排引擎。在默认情况下，没有选择不同的容器编排引擎，当前环境会使用cattle。在host被添加都Rancher中后，你可以开始添加[services]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-services/)或者从[Rancher catalog]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/catalog/)启动一个模版应用。

<a id="single-container-external-database"></a>

### 启动 Rancher Server - 单容器部署 - 使用外部数据库

除了使用内部的数据库，你可以启动一个Rancher server并使用一个外部的数据库。启动命令与之前一样，但添加了一些额外的参数去说明如何连接你的外部数据库。

> **Note:** 在你的外部数据库中，数据库名和数据库用户需要提前创建。但不需要创建数据库的schema。Rancher为自动创建Rancher所需要的schemas。

以下是创建数据库和数据库用户的SQL命令例子

```sql
> CREATE DATABASE IF NOT EXISTS cattle COLLATE = 'utf8_general_ci' CHARACTER SET = 'utf8';
> GRANT ALL ON cattle.* TO 'cattle'@'%' IDENTIFIED BY 'cattle';
> GRANT ALL ON cattle.* TO 'cattle'@'localhost' IDENTIFIED BY 'cattle';
```

启动一个Rancher连接一个外部数据库，你需要在启动容器的命令中添加额外参数。

```bash
$ sudo docker run -d --restart=unless-stopped -p 8080:8080 rancher/server \
    --db-host myhost.example.com --db-port 3306 --db-user username --db-pass password --db-name cattle
```

大部分的输入参数都有默认值并且是可选的，只有MySQL server的地址是必须输入的。

```bash
--db-host               IP or hostname of MySQL server
--db-port               port of MySQL server (default: 3306)
--db-user               username for MySQL login (default: cattle)
--db-pass               password for MySQL login (default: cattle)
--db-name               MySQL database name to use (default: cattle)
```

<br>

> **Note:** 在之前版本的Rancher server中，我们需要使用环境变量去连接外部数据库。这些环境变量会继续生效，单Rancher建议使用命令参数代替。

<a id="single-container-bind-mount"></a>

### 启动 Rancher Server - 单容器部署 - 挂载MySQL数据库的数据目录

如果你在你的容器中想持久化你的数据库到一个host上的卷，可以在启动Rancher时挂载MySQL的数据卷。

```bash
$ sudo docker run -d -v <host_vol>:/var/lib/mysql --restart=unless-stopped -p 8080:8080 rancher/server
```
使用这条命令，数据库就会持久化在host上。如果你有一个现有的Rancher server容器并且想挂在MySQL的数据卷，可以参考以下的介绍[upgrading documentation]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/upgrading/#single-container-bind-mount)。

<a id="multi-nodes"></a>

### 启动 Rancher Server - 多节点的HA部署

Running Rancher server in High Availability (HA) is as easy as running [Rancher server using an external database](#using-an-external-database), exposing an additional port, and adding in an additional argument to the command for the external load balancer.
在高可用(HA)的模式下运行Rancher server与运行[Rancher server 使用外部数据库](#using-an-external-database)一样简单，需要暴露一个额外的端口，添加额外的参数到启动命令中，并且运行一个外部的负载均衡就可以了。

#### HA部署需求

* HA 节点:
    * 所有现代Linux分发版并且安装了[supported version of Docker]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/#supported-docker-versions)。[RancherOS](http://docs.rancher.com/os/), Ubuntu, RHEL/CentOS 7 都是经过严格的测试。
	  * 对于 RHEL/CentOS, 默认的 storage driver, 例如 devicemapper using loopback, 并不被[Docker](https://docs.docker.com/engine/reference/commandline/dockerd/#/storage-driver-options)推荐。 请参考Docker的文档去修改使用其他的storage driver。
	  * 对于 RHEL/CentOS, 如果你想使用 SELinux, 你需要 [安装额外的 SELinux 组件]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/installing-rancher/selinux/).
    * `9345`, `8080` 端口需要在各个节点之间能够互相访问
    * 1GB RAM
* MySQL database
    * 至少 1 GB RAM
    * 每个Rancher server节点需要50个连接 (例如：3个节点的Rancher则需要至少150个连接)
    * MYSQL Configuration Requirements
      * Option 1: Run with Antelope with default of `COMPACT`
      * Option 2: Run MySQL 5.7 with Barracuda where the default `ROW_FORMAT` is `Dynamic`
* External Load Balancer
    * 负载均衡服务器需要能访问Rancher server节点的 `8080` 端口

> **Note:** 目前Rancher中并不支持Docker for Mac

#### 大规模部署建议

* 每一个Rancher server节点需要有4 GB 或者8 GB的堆空间，意味着需要8 GB或者16 GB内存
* MySQL数据库需要有快速磁盘
* 对于一个完整的HA，建议使用一个有副本的Mysql数据库。使用Galera并强制写入一个mysql节点（由于事务锁的原因，强制写入一个节点可以提高数据库集群效率）将会是另外一种选择。

1. 在每个需要加入Rancher server HA集群的节点上，运行以下命令：

   ```bash
   # Launch on each node in your HA cluster
   $ docker run -d --restart=unless-stopped -p 8080:8080 -p 9345:9345 rancher/server \
        --db-host myhost.example.com --db-port 3306 --db-user username --db-pass password --db-name cattle \
        --advertise-address <IP_of_the_Node>
   ```

   在每个节点上，`<IP_of_the_Node>` 需要在每个节点上唯一，因为这个IP会被添加到HA的设置中。

   如果你修改了 `-p 8080:8080` 并在host上暴露了一个不一样的端口，你需要添加 `--advertise-http-port <host_port>` 参数到命令中。

   > **Note:** 你可以使用 `docker run rancher/server --help` 获得命令的帮助信息

2. 配置一个外部的负载均衡器将会在端口 `80` 以及 `443` 中转发流量到运行Ranher server节点的 `8080`节点中。负载均衡器必须支持websockets 以及 forwarded-for 的Http请求头以支持Rancher的功能。参考 [SSL settings page]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}//installing-rancher/installing-server/basic-ssl-config/) 这个配置的例子。

#### Notes on the Rancher Server Nodes in HA

如果你的Rancher server节点上的IP修改了，你的节点将不再存在于Rancher HA集群中。你必须停止在`--advertise-address`配置了不正确IP的Rancher server容器并启动一个使用正确IP地址的Rancher server的容器。

<a id="elb"></a>

### 使用AWS的Elastic/Classic Load Balancer作为Rancher Server HA的负载均衡器

我们建议使用AWS的ELB作为你Rancher server的负载均衡器。为了让ELB与Rancher的websockets正常工作，你需要开启proxy protocol模式并且保证HTTP support被停用。 默认的，ELB是在HTTP/HTTPS模式启用，在这个模式下不支持websockets。listener的配置需要被特别的关注。

如果你在配置ELB中遇到问题，我们建议你参考[terraform version](#configuring-using-terraform)。

> **Note:** 如果你正在使用自签名的证书, 请参考 how to [configure your ELB in AWS under our SSL section]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/installing-rancher/installing-server/basic-ssl-config/#elb).

#### Listener 配置 - Plaintext

简单的来说，使用非加密的负载均衡用途，需要以下的listener配置：

| Configuration Type | Load Balancer Protocol | Load Balancer Port | Instance Protocol | Instance Port |
|---|---|---|---|---|
| Plaintext | TCP | 80 | TCP | 8080  (或者使用启动Rancher server时 `--advertise-http-port` 指定的端口) |

#### 启用 Proxy Protocol

In order for websockets to function properly, the ELB proxy protocol policy must be applied.
为了使websockets正常工作，ELB的proxy protocol policy必须被启用。

* 启用 [proxy protocol](http://docs.aws.amazon.com/ElasticLoadBalancing/latest/DeveloperGuide/enable-proxy-protocol.html) 模式

```
$ aws elb create-load-balancer-policy --load-balancer-name <LB_NAME> --policy-name <POLICY_NAME> --policy-type-name ProxyProtocolPolicyType --policy-attributes AttributeName=ProxyProtocol,AttributeValue=true
$ aws elb set-load-balancer-policies-for-backend-server --load-balancer-name <LB_NAME> --instance-port 443 --policy-names <POLICY_NAME>
$ aws elb set-load-balancer-policies-for-backend-server --load-balancer-name <LB_NAME> --instance-port 8080 --policy-names <POLICY_NAME>
```

* Health check可以配置使用HTTP:8080下的 `/ping` 路径进行健康检查

#### 使用Terraform进行配置

以下是使用Terraform配置的例子：

```
resource "aws_elb" "lb" {
  name               = "<LB_NAME>"
  availability_zones = ["us-west-2a","us-west-2b","us-west-2c"]
  security_groups = ["<SG_ID>"]

  listener {
    instance_port     = 8080
    instance_protocol = "tcp"
    lb_port           = 443
    lb_protocol       = "ssl"
    ssl_certificate_id = "<IAM_PATH_TO_CERT>"
  }

}

resource "aws_proxy_protocol_policy" "websockets" {
  load_balancer  = "${aws_elb.lb.name}"
  instance_ports = ["8080"]
}
```

<a id="alb"></a>

### 使用AWS的Application Load Balancer(ALB) 作为Rancher Server HA的负载均衡器

我们不再推荐使用AWS的Application Load Balancer (ALB)替代Elastic/Classic Load Balancer (ELB)。如果你依然选择使用ALB，你需要直接指定流量到Rancher server节点上的HTTP端口，默认是8080。


<a id="ldap"></a>

### TLS认证使用AD/OpenLDAP

为了在Rancher server上启用Active Directory或OpenLDAP并使用TLS，Rancher server容器在启动的时候需要配置LDAP证书，证书是LDAP服务提供方提供。证书保存在需要运行Rancher server的Linux机器上。

启动Rancher并挂载证书。证书在容器内部 **必须** 命名为`ca.crt`。

```bash
$ sudo docker run -d --restart=unless-stopped -p 8080:8080 \
  -v /some/dir/cert.crt:/var/lib/rancher/etc/ssl/ca.crt rancher/server
```

你可以使用Rancher server的日志检查传入的 `ca.crt` 证书是否生效

```bash
$ docker logs <SERVER_CONTAINER_ID>
```

在日志的开头，会显示正式已经被正确家在的确认信息。

```bash
Adding ca.crt to Certs.
Updating certificates in /etc/ssl/certs... 1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d....done.
Certificate was added to keystore
```

<a id="http-proxy"></a>

### 在HTTP代理后方启动 Rancher Server

为了设置HTTP Proxy，Docker daemon需要修改配置并指向这个代理。在启动Rancher server前，需要编辑配置文件 `/etc/default/docker` 添加你的代理信息并重启Docker服务。

```bash
$ sudo vi /etc/default/docker
```

在文件中，编辑 `#export http_proxy="http://127.0.0.1:3128/"` 并修改它指向你的代理。保存修改并重启Docker。重启Docker的方式在每个OS上都不一样。

> **Note:** If you are running Docker with systemd, please follow Docker's [instructions](https://docs.docker.com/articles/systemd/#http-proxy) on how to configure the HTTP proxy.
> **Note:** 如果你使用systemd运行Docker, 请参考Docker官方的文档 [instructions](https://docs.docker.com/articles/systemd/#http-proxy) 去配置http proxy设置。

为了使得[Rancher catalog]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/catalog/)加载正常，HTTP代理设置必须在Rancher server运行的环境变量中。

```bash
$ sudo docker run -d \
    -e http_proxy=<proxyURL> \
    -e https_proxy=<proxyURL> \
    -e no_proxy="localhost,127.0.0.1" \
    -e NO_PROXY="localhost,127.0.0.1" \
    --restart=unless-stopped -p 8080:8080 rancher/server
```

如果[Rancher catalog]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/catalog/) 不会被使用，则使用你平常的Rancher server命令即可。

当向Rancher添加主机时[adding hosts]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/) ，在HTTP代理中不需要额外的设置和要求。
