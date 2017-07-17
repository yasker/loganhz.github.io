---
title: Installing Rancher Server with No Internet Access
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
redirect_from:
  - /rancher/installing-rancher/installing-server/no-internet-access/
---

## 内网启动Rancher
---

不可对外访问的网络环境（内网）也是可以启动 Rancher 服务的。在这种拓扑下，可以通过内网提供的IP或者域名来访问Rancher的操作界面（UI界面）。另外，也可以用HTTP代理或者私有镜像库来配置 Rancher。

需要注意的是，在内网中启动一个 Rancher 服务会导致一些特性无效，比如：

* 使用操作界面来启动云公有云提供商（例如AWS，DigitalOcean，阿里云，vSphere等）提供的主机。只能添加 [custom hosts（自定义主机）]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/custom/) 来初始化Rancher；
* Github 授权认证。

### 前提条件

为了支持这种拓扑，有些 [前提条件]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/installing-rancher/installing-server/#requirements) 是必须要满足的。

### Rancher服务的标签类型

每次发布主版本（major release）时，Rancher 除了会提供对应版本号的文档外，还会提供 _两个_不同的镜像描述标签（tags）:

* `rancher/server:latest` 表示最新的版本，这类版本虽然都通过Rancher CI（持续集成）的检验，但并不推荐在生产环境中部署；
* `rancher/server:stable` 表示稳定的版本，这类版本推荐在生产环境中部署。

另外，以`rc{n}`结尾的发布版本，都是 Rancher 研发团队测试所用的构建版本，请直接忽略它们。

### 使用私有镜像仓库

假设内网已经存在私有镜像仓库，或类似的支持分布式 Docker 镜像管理的服务。如果还没有，可以浏览 Docker 官网提供的 [私有镜像仓库](https://docs.docker.com/registry/) 文档来搭建，再此不再累述.

#### 推镜像给仓库

在安装或升级Rancher服务之前，**必须保证**对应版本号的所有镜像（例如： `rancher/server`，`rancher/agent`，以及任何 [基础服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/) 涉及的镜像) 都必须上传到私有仓库或者类似的服务中。如果这些镜像不存在或者版本信息不对，Rancher服务讲无法正常完成安装或升级。

对于每次发布的 Rancher 服务（`rancher/server`），对应的 Rancher 代理（`rancher/agent`）和 Rancher 代理实例（`rancher/agent-instance`）的镜像版本信息，都在发布记录中摘记。而针对其他基础服务用到的镜像版本信息，就需要查看 [官方模板](https://github.com/rancher/rancher-catalog) 的`infra-templates`目录和 [社区模板](https://github.com/rancher/community-catalog) 来获取所关联的镜像。如果用到 [应用商店]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/catalog/)，还需要关注具体应用使用的docker-compose.yml来获取镜像版本信息。

##### 使用命令行推镜像给仓库

在这个例子中，假设某台机器可以同时访问私有镜像仓库和 DockerHub。首先从 DockerHub 中拉取 `rancher/server` 和 `rancher/agent` 的镜像，然后对拉取下来的镜像做标签私仓化处理，最后再推送到私有镜像仓库。一种推荐的做法是，私有镜像仓库的镜像的版本信息对照于 DockerHub 的版本信息。

```bash
# rancher/server
$ docker pull rancher/server:v1.6.0
$ docker tag rancher/server:v1.6.0 localhost:5000/<NAME_OF_LOCAL_RANCHER_SERVER_IMAGE>:v1.6.0
$ docker push localhost:5000/<NAME_OF_LOCAL_RANCHER_SERVER_IMAGE>:v1.6.0

# rancher/agent
$ docker pull rancher/agent:v1.1.3
$ docker tag rancher/agent:v1.1.3 localhost:5000/<NAME_OF_LOCAL_RANCHER_AGENT_IMAGE>:v1.1.3
$ docker push localhost:5000/<NAME_OF_LOCAL_RANCHER_AGENT_IMAGE>:v1.1.3
```

<br>

> **提示:** 对于任何基础服务镜像, 可以按照以下的步骤来获取。

#### 通过私有镜像仓库启动Rancher服务

在上文中描述的机器中，启动 Rancher 服务以后，就可以使用特定的 Rancher 代理镜像了。一种推荐的做法是，使用镜像时使用具体的版本号码代替`latest`来指代版本信息。

例如:

```bash
$ sudo docker run -d --restart=unless-stopped -p 8080:8080 \
    -e CATTLE_BOOTSTRAP_REQUIRED_IMAGE=<Private_Registry_Domain>:5000/<NAME_OF_LOCAL_RANCHER_AGENT_IMAGE>:v1.1.3 \
    <Private_Registry_Domain>:5000/<NAME_OF_LOCAL_RANCHER_SERVER_IMAGE>:v1.6.0
```

#### Rancher操作界面

默认情况下，操作界面访问（含接口API）是通过 `8080` 端口暴露，可以用以下这个地址访问：`http://<SERVER_IP>:8080`。

#### 添加主机

在操作界面，点击 **Add Host（添加主机）**后，就会进入 **Host Registration（主机登记）** 界面。点击一下 **Save（保存）**后即可添加主机。

这个时候由于没法使用公有云提供商的主机服务，所以请点击 **Custom（自定义）**图标来增加主机。

操作界面上生成的添加指令，在某台集群管控节点主机（运行一个`rancher/agent`容器的主机）执行时，将启动来自私有镜像仓库的 Rancher 代理镜像。

##### 一个由操作界面生成的添加指令例子

```bash
$ sudo docker run -d --privileged -v /var/run/docker.sock:/var/run/docker.sock <Private_Registry_Domain>:5000/<NAME_OF_LOCAL_RANCHER_AGENT_IMAGE>:v1.1.3 http://<SERVER_IP>:8080/v1/scripts/<security_credentials>
```

#### 为基础服务栈配置默认的仓库

默认在 Rancher 中, 所有 [基础服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/) 都默认从 DockerHub上拉取镜像。可以通过 **API settings（接口设置）**改变默认的的仓库，比如改成私有的镜像仓库。

* **添加一个私有镜像仓库：** 在 **Infrastructure（基础架构）** 中选择 **Registries（仓库）**，添加可以为基础架构提供镜像的仓库服务。

* **更新默认的仓库：** 在 **Admin（管理）** 的 **Setting（设置）** 的 **Advanced Settings（高级设置）**里，点击 **I understand that I can break things by changing advanced settings（已经了解修改高级设置后可能造成意外影响）**。找到 **registry.default（默认仓库）** 的设置项并点击修改按钮，修改为私有镜像仓库的地址，点击 **Save（保存）**。一旦 `registry.default` 的值被更新了，基础架构服务将从这个地址上获取镜像。

* **创建一个新环境：** 当默认的镜像仓库地址被改变后，需要重新创建环境以便于基础架构服务可以从使用这个新的镜像仓库地址。原来旧的环境中，基础架构服务仍然指向DockerHub。

> **注意：** 原来旧的环境中的基础架构栈，仍然使用原来默认的镜像仓库地址（例如，在出厂设置中就是 DockerHub，那就一直都是 DockerHub）。只有把栈删除了，然后重新启动才能试用新的镜像仓库地址。可以通过 **Catalog（商店）** 中的 **Library（实验室）**进行部署。

### 使用HTTP代理

再次提醒，在启动了 Rancher 服务以后，只能通过内网来访问 Rancher 的操作界面。

#### 通过HTTP代理来配置Docker

通过修改 Rancher 服务和 Rancher 代理的 Docker daemon（Docker的守护进程）的配置文件`/etc/default/docker`，使其指向对应的HTTP代理地址，重启 Docker daemon 之后，即可启用HTTP代理。

```bash
$ sudo vi /etc/default/docker
```

打开配置文件后，修改或增加 `export http_proxy="http://${YOUR_PESONAL_REGISTYR_ADDRESS}/"`。然后保存它并重启 Docker deamon。不同的操作系统重启 Docker deamon（也就是重启 Docker ）的方法是不一样的，请自行了解，不再累述。

> **注意：** 如果Docker是通过systemd来运行的，那么可以参考[这篇文章](https://docs.docker.com/articles/systemd/#http-proxy)来了解更多。

#### 启动Rancher服务

使用HTTP代理的时候，Rancher服务不需要使用任何环境变量即可启动。所以，启动操作就和没有使用代理服务是一样的，指令如下：

```bash
sudo docker run -d --restart=unless-stopped -p 8080:8080 rancher/server
```

#### Rancher操作界面

默认情况下，操作界面访问（含接口API）是通过 `8080` 端口暴露，可以用以下这个地址访问：`http://<SERVER_IP>:8080`。

#### 添加主机

在操作界面，点击 **Add Host（添加主机）**后，就会进入 **Host Registration（主机登记）** 界面。点击一下 **Save（保存）**后即可添加主机。

这个时候由于没法使用公有云提供商的主机服务，所以请点击 **Custom（自定义）**图标来增加主机。

操作界面上生成的添加指令，在某台集群管控节点主机（运行一个`rancher/agent`容器的主机）执行时，将启动来自私有镜像仓库的 Rancher 代理镜像。
