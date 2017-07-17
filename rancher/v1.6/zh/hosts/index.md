---
title: Hosts in Rancher
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---

## 入门指南
---

在 Rancher 中，主机（Host）是调度资源的基本单位（直观的理解就是所发生的操作最终都会落到某台主机上），它可以是虚拟的或者物理的Linux服务器。Rancher管理的主机需要满足以下的条件：


* 任何[可以运行Docker](#supported-docker-versions)的 Linux 发行版本，例如：[RancherOS](http://docs.rancher.com/os/)，Ubuntu，RHEL/CentOS 7。不过针对RHEL/CentOS系列，有些需要注意的地方：
    * Docker 并不推荐使用 RHEL/CentOS 默认的存储驱动（devicemapper），可以参考[这篇文档](https://docs.docker.com/engine/reference/commandline/dockerd/#/storage-driver-options)来修改；
    * 如果启用 SELinux，[需要安装额外的模块]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/installing-rancher/selinux/)；
    * 内核版本要求是 `3.10.0-514.2.2.el7.x86_64` 及以上，建议使用 RHEL/CentOS 7.3 或者更高的发行版本。
* 1GB的内存；
* 推荐 w/AES-NI 架构的 CPU；
* 支持使用 http 或 https 访问 Rancher 服务，默认端口是8080；
* 主机与主机之间是可以相互访问的，从而确保 Rancher 可以跨主机对容器进行管理。

另外，Rancher 也可以管理由 Docker Machine 驱动的主机，只要这些主机满足上面的条件即可。

在 Rancher 的操作界面上，选择 **Infrastructure（基础架构）-> Hosts（主机）**，点击 **Add Host（添加主机）**按钮，然后再做些工作，就可以让 Rancher 管理到这台新主机了。

### Docker版本适用对比

版本               | Rancher适用？ | K8S适用？ | 安装脚本 |
----------------------|------------|---------------------|-----------------
`1.9.x` and earlier   | No         |                     |
`1.10.0` - `1.10.2`   | No         |                     |
`1.10.3` (and higher) | **Yes**    | **Yes**             | `curl https://releases.rancher.com/install-docker/1.10.sh | sh`
`1.11.x`              | No         |                     | `curl https://releases.rancher.com/install-docker/1.11.sh | sh`
`1.12.0` - `1.12.2`   | No         |                     |
`1.12.3` (and higher) | **Yes**    | **Yes**             | `curl https://releases.rancher.com/install-docker/1.12.sh | sh`
`1.13.x`              | **Yes**    | No                  | `curl https://releases.rancher.com/install-docker/1.13.sh | sh`
`17.03.x-ee`          | **Yes**    | No                  | n/a
`17.03.x-ce`          | **Yes**    | No                  | `curl https://releases.rancher.com/install-docker/17.03.sh | sh`
`17.04.x-ce`          | No         |                     | `curl https://releases.rancher.com/install-docker/17.04.sh | sh`
`17.05.x-ce`          | No         | No                  |

### 安装特定版本的Docker

一般会使用 `curl https://get.docker.com | sh` 脚本来安装最新版的 Docker 。但是，最新版的 Dokcer 有可能不适用于正准备安装或已经在使用中的 Rancher 版本。因此，一种推荐的做法是：安装特定版本的 Docker。按照上方的对比表，选择 Rancher 适用的安装脚本执行即可。

> **注意：** 如果从操作界面上添加主机，可以通过 **Advanced Options（高级选项）** 里面的 **Docker Install URL（Docker安装路径）**来选择需要安装的 Docker 版本。

### 主机是如何工作的？

新添加的主机是通过启动 Rancher agent（代理）容器来和 Rancher server（服务）进行联接沟通的。可以通过操作界面的 **Add Host（添加主机）**，选择 **Custom（自定义）**，Rancher 服务会生成一个注册令牌以及接口访问密钥。注册令牌是代理与 服务在连接时使用，而接口访问密钥就是连接成功后调用后续接口的认证和授权。

从设计的角度而言，Rancher 代理是运行在独立于 Rancher 服务的主机上，所以代理本身是不可信的。采用接口访问密钥的机制，可以确保代理只访问被授权的接口，而服务只响应可信的请求。但是目前是单向认证，只认证从代理到服务的请求，并没有认证从服务到代理的响应。因此，用户可以根据需要，使用TLS和证书来做校验。

每个[环境]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/)的注册令牌，都是由 Rancher 服务生成并保存到数据库，然后和接口访问密钥一起下发给代理使用。代理和服务之间是采用AES对称加密的点对点通讯。

<a id="addhost"></a>

### 添加主机

第一次添加主机时，Rancher 服务会要求配置 [Host Registration URL（主机注册地址）]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/settings/#host-registration)。这个地址可以是域名或者IP地址（如果80端口不可访问，还需要加上可访问的端口号，默认 `8080`），能够访问 Rancher 接口即可。任何时候都可以改变 [主机注册地址]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/settings/#host-registration)，相关操作可以查看 **Admin（系统管理）** 下的 **Settings（系统设置）**。设置好主机注册地址后，点击 **Save（保存）**.

假设添加的是来自云提供商（例如AWS，DigitalOcean，阿里云，vSphere等）所提供的主机或者本地（例如VirtualBox，VMWare等）设置好的主机。对于云提供商，Rancher 是通过 `docker-machine` 来添加的，所以本质上实现了 Docker Machine 驱动的厂商的云主机，都可以被添加。

接下来，选择:

* [添加自定义主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/custom/)
* [添加 AWS EC2 主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/amazon/)
* [添加 Azure 主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/azure/)
* [添加 DigitalOcean 主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/digitalocean/)
* [添加 Exoscale 主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/exoscale/)
* [添加 Packet 主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/packet/)
* [添加 Rackspace 主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/rackspace/)
* [添加其他云提供商的主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/other/)

当主机被添加到 Rancher 时，这台主机会运行一个合适版本的 `rancher/agent` 容器。

<a id="labels"></a>

### 主机标签

在 Rancher 中，可以通过添加标签的方式来管理某台主机，做法就是在 `rancher/agnet` 容器启动的时候，以环境变量的方法把标签加进去。在操作界面上可以看到，标签其实是一些唯一的键值。值得注意的是，如果有两个相同的键但是值不一样，那么最后添加的值将会被 Rancher 使用。

给主机增加标签时，可以根据需求使用 [调度服务或负载均衡]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/scheduling/) 的标签。如果不希望某个服务或者要求某个服务必须运行在某台主机上，可以 [在添加一个服务时]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-services/) 进行配置。

如果需要使用 [外部 DNS 服务（类似 Bind9 这类）]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/external-dns-service/) 以及 [通过DNS-IP映射来管理不在 Rancher 内启动的服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/external-dns-service/#using-a-specific-ip-for-external-dns)，那就要在运行 `rancher/agent` 时增加标签 `io.rancher.host.external_dns_ip=<IP_TO_BE_USED_FOR_EXTERNAL_DNS>`。切记，当要某个容器服务要使用外部 DNS 服务时，一定要增加这个标签。

通过操作界面，在添加云提供商的主机时添加的标签，`rancher/agent` 会保证这些标签都会自动作用到后续调度到这台主机内的容器服务。

如果通过操作界面添加自定义主机，当增加标签时，界面上的运行注册脚本会对应的增加环境变量：`CATTLE_HOST_LABELS`。比如，增加一个标签：foo=bar，会出现下面的效果：

```bash
$  sudo docker run -e CATTLE_HOST_LABELS='foo=bar' -d --privileged \
    -v /var/run/docker.sock:/var/run/docker.sock rancher/agent:v0.8.2 \
    http://<rancher-server-ip>:8080/v1/projects/1a5/scripts/<registrationToken>

# 当再增加一个标签：hello=world
$  sudo docker run -e CATTLE_HOST_LABELS='foo=bar&hello=world' -d --privileged \
    -v /var/run/docker.sock:/var/run/docker.sock rancher/agent:v0.8.2 \
    http://<rancher-server-ip>:8080/v1/projects/1a5/scripts/<registrationToken>
```

<br>

> **注意：** `rancher/agent` 的版本与 `rancher/server` 的版本是相关的，执行添加自定义主机的时候，需要注意注册脚本的 `rancher/agent` 是否正确。正常情况下，通过操作界面获取的脚本内的版本信息都是正确的，用户不需要做额外修改。

#### 自动添加的标签

Rancher 会自动创建一些和 Linux 内核版本信息以及 Docker 引擎版本信息相关的标签。

键 | 值 | 描述
----|----|----
`io.rancher.host.linux_kernel_version` | Linux内核版本 (e.g, `3.19`) |  主机当前运行的内核版本
`io.rancher.host.docker_version` | Docker引擎版本 (e.g. `1.10`) | 主机运行的 Docker 版本
`io.rancher.host.provider` | 云提供商信息 | 目前仅适用于AWS
`io.rancher.host.region` | 云提供商地域 | 目前仅适用于AWS
`io.rancher.host.zone` | 云提供商区域 | 目前仅适用于AWS

### 主机调度地址

为了 Rancher [能够对多台主机上进行调度]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/scheduling/#scheduling-against-multiple-ips-of-a-host)，需要给这些主机进行配置。配置的方法就是给已经存在的主机（运行了 `rancher/agent` 容器的服务器）或者新添加的主机（还没有运行 `rancher/agent` 容器的服务器）增加调度主机所在的地址。

#### 对已在调度中的主机增加新调度地址
在环境中已存在的主机，通过增加 `io.rancher.scheduler.ips` 标签来提供其他 `rancher/server` 对其的调度。通过操作界面，点击这台主机的 **Edit Host（主机编辑）** 按钮，然后增加 **Scheduler IP（调度IP）**。如果是通过接口的方式，只需要给主机添加标签 `io.rancher.scheduler.ips` 和值（多个地址，可以通过逗号分隔）即可。

> **注意：** 在没有添加调度地址前，如果已经调度执行了某些容器服务，那么这些容器服务默认被认为是 `0.0.0.0` 的地址做的调度。那就意味着，后续添加的调度地址也可以对这些容器服务做管理。

#### 对新主机的添加调度地址

对于新添加的 [自定义主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/custom/) 需要像下面的例子，给注册脚本增加一个环境变量 `CATTLE_SCHEDULER_IPS` ：

```bash
$  sudo docker run -e CATTLE_SCHEDULER_IPS='1.2.3.4,<IP2>,..<IPN>' -d --privileged \
    -v /var/run/docker.sock:/var/run/docker.sock rancher/agent:v0.8.2 \
    http://<rancher-server-ip>:8080/v1/projects/1a5/scripts/<registrationToken>
```

### 在代理后的主机

如果当前的环境是在代理之后，要给 Rancher 添加主机，需要修改这台主机的 Docker deamon 指向代理。相关的细节可以浏览 [本文]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/custom/#hosts-behind-a-proxy)，在此不在累述。

<a id="machine-config"></a>

### 访问云提供商的主机

当用 Rancher 添加云提供商的主机时，实质上是采用 Docker Machine 执行的工作。

### 克隆主机

在云提供商上启动主机需要使用访问密钥，Rancher 提供了克隆的方法，来轻松地创建另一个主机，而无需再次输入所有认证配置。在操作界面上，从 **Infrastructure（基础架构）** 进入 **Hosts（主机）** 页面，点击某台主机的下拉菜单，选择 **Clone（克隆）**，然后就会进入之前的认证配置都已经填写的 **Add Host（添加主机）** 页面。

### 修改主机

在操作界面上，从 **Infrastructure（基础架构）** 进入 **Hosts（主机）** 页面，点击需要修改的主机的下拉菜单，选择 **Edit（编辑）**，就可以修改这台主机的名称，描述以及标签。

### 启停主机

停止一台主机后，操作界面上会显示 _Inactive_ 状态。在这种状态下，不会再有容器服务被部署到这台主机。而处于 _Active_ 状态下的主机，容器服务会被正常的部署、停止、重启或销毁。

如果需要停止一台主机，从 **Infrastructure（基础架构）** 进入 **Hosts（主机）** 页面，点击需要停止的这台主机的下拉菜单，选择 **Deactivate（停止）** 即可。

如果需要把一台停止的主机重新激活，从 **Infrastructure（基础架构）** 进入 **Hosts（主机）** 页面，点击已经停止的这台主机的下拉菜单，选择 **Activate（激活）** 即可。

> **注意：** 在 Rancher 中如果主机宕机了（比如处于 `reconnecting` 或 `inactive` 的状态），需要进行 [健康检查]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/health-checks/) 以便于 Rancher 把这台宕掉的主机上的容器服务迁移到其他主机上执行。

### 在Rancher内删除主机

在 Rancher 内删除主机的操作需要进行几个步骤：从 **Infrastructure（基础架构）** 进入 **Hosts（主机）** 页面，点击需要删除的主机的下拉菜单，选择 **Deactivate（停止）**。当主机完成停止以后，将会显示 _Inactive_ 状态。然后点击下拉菜单，选择 **Delete（删除）**，Rancher 会执行对这台主机的删除操作。当显示 _Removed_ 状态时，就表示这台主机已经被删除了。但是，仍然可以在操作界面上看到这台主机，只有当点击下拉菜单，选择 **Purge（清理）**后，这台主机才会从操作界面上消失。

如果这台主机是由 Rancher 调用 `docker-machiine` 基于云提供商的驱动创建，按照上述的删除操作执行后，被删除的主机也会在云提供商的管理界面中消失。但是，如果是采用 [添加自定义主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/custom/) 的方式所添加的云提供商主机，被删除的主机还会在云提供商的管理界面中被查看到。而且这台主机内的容器服务（例如 `rancher/agent`）还是保留着的。可以认为通过自定义添加的云提供商的主机被删除后，只是从 Rancher 的调度中解离出去，但是它原来的生命周期 Rancher 不会干涉。

### 在Rancher外删除主机

在 Rancher 外删除主机，也就是意味着不是按照 Rancher 操作界面或者API来删除主机。最简单的例子就是，在 Rancher 集群内，有一台云提供商提供的主机。通过云提供商的管理界面删除了这台主机，这个删除行为 Rancher 是无法感知的。Rancher 一直尝试重新连接这台已经删除的主机，并显示 _Reconnecting_ 的状态。 因此，为了同步回来删除的状态，还需要从 **Infrastructure（基础架构）** 进入 **Hosts（主机）** 页面，点击已经删除的主机的下拉菜单，选择 **Delete（删除）**。
