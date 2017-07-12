---
title: Rancher Compose
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
redirect_from:
  - /rancher/rancher-compose/
---

## Rancher Compose
---

Rancher compose 是一个多主机版本的 Docker Compose.  它运行于Rancher UI 里属于一个[环境(environment)]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/)多个[主机(hosts)]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/)的[栈(stack)]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/stacks/)里.Rancher Compose 启动的容器会被部署在满足[调度规则(scheduling rules)]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/scheduling/)的同一环境中的任意主机里. 如果没有设置调度规则，那么这些服务容器会被调度至最少容器运行的主机上运行. 这些被 Rancher Compose 启动的容器的运行效果是和在 UI 上启动的效果是一致的.

Rancher Compose 工具的工作方式是跟 Docker Compose 的工作方式是相似的，并且兼容版本V1和V2的 `docker-compose.yml` 文件。为了启用 Rancher 的特性，您需要额外一份`rancher-compose.yml`文件，这份文件扩展并覆盖了`docker-compose.yml`文件。例如，服务缩放和健康检查这些功能就会在`rancher-compose.yml`中体现。

在阅读这份Rancher Compose文档之前，我们希望您已经懂得 `Docker Compose` 了。如果您还不认识 Docker Compose，请先阅读 [Docker Compose](https://docs.docker.com/compose/)文档。

### 安装

Rancher Compose 的可执行文件下载链接可以在 UI 的右下角中找到，我们为您提供了Windows, Mac 以及 Linux 版本供您使用。

另外，您也可以到[Rancher Compose的发布页](https://github.com/rancher/rancher-compose/releases) 找到可执行二进制文件的下载链接。

### 为 Rancher Compose 设置 Rancher server

为了让 Rancher Compose 可以在 Rancher 实例中启动服务，您需要设置一些环境变量或者在Rancher Compose命令中送一些参数。必要的环境变量分别是 `RANCHER_URL`, `RANCHER_ACCESS_KEY`, 以及 `RANCHER_SECRET_KEY`。 access key 和 secret key是一个环境API Keys, 可以在**API** -> **高级选项**菜单中创建得到。

> **注意:** 默认情况下，在**API**菜单下创建的是账号API Keys, 所以您需要在**高级选项**中创建环境API Keys.

```bash
# Set the url that Rancher is on
$ export RANCHER_URL=http://server_ip:8080/
# Set the access key, i.e. username
$ export RANCHER_ACCESS_KEY=<username_of_environment_api_key>
# Set the secret key, i.e. password
$ export RANCHER_SECRET_KEY=<password_of_environment_api_key>
```

如果您您不想设置环境变量，那么您需要在Rancher Compose 命令中手动送入这些变量：

```bash
$ rancher-compose --url http://server_ip:8080 --access-key <username_of_environment_api_key> --secret-key <password_of_environment_api_key> up
```

<br>

现在您可以使用Rancher Compose 配合`docker-compose.yml`文件来[启动服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-services/#adding-services-with-rancher-compose)了。这些服务会在环境API keys对应的[环境]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/)中启动服务的。

就像Docker Compose，您可以在命令后面加上服务名称来选择启动全部或者仅启动指定某些`docker-compose.yml`中服务

```baseurl
$ rancher-compose up servicename1 servicename2
$ rancher-compose stop servicename2
```

### 调试 Rancher Compose

您可以设置环境变量`RANCHER_CLIENT_DEBUG`的值为`true`来让Rancher Compose输出所有被执行的CLI 命令。

```bash
# Print verbose messages for all CLI calls
$ export RANCHER_CLIENT_DEBUG=true
```

<br>

如果你不需要所有的 CLI 命令信息，您可以在命令后上`--debug`来指定输出哪些可视化 CLI 命令。

```bash
$ rancher-compose --debug up -d
```

### 删除服务或容器

在缺省情况下，Rancher Compose不会删除任何服务或者容器。这意味着如果您在一行命令里执行两次 `up` 命令，那么第二个 `up` 命令不会起任何作用。这是因为第一个 `up` 命令会创建出所有东西后让他们自己运行。即使您没有在 `up` 中使用 `-d` 参数，Rancher Compose 也不会删除您任何服务。为了删除服务，您只能使用 `rm` 命令。

### 构建

构建 docker 镜像可以有两种方法。第一种方法是通过给 build 命令一个 git 或者 http URL参数来利用远程资源构建，另一种方法则是让 build 利用本地目录，那么会上传构建上下文到 S3 并在需要时在各个节点执行

为了可以基于S3来创建，您需要[设置 AWS 认证](https://github.com/aws/aws-sdk-go/#configuring-credentials)。我们提供了一个说明怎样利用在Rancher Compose 里使用S3[详细例子]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/rancher-compose/build/)供您参考
