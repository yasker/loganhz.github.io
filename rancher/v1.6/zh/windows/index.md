---
title: Windows in Rancher
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---

## Windows (实验性)
---

你需要创建一个新 [环境]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/)，并且把这个环境的 [环境模版]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/#what-is-an-environment-template)的容器编排平台设为 **Windows**，才能在 Rancher 中部署 Windows。

目前 Rancher 只支持在特定主机上创建容器。大多数在 Cattle 和 Rancher UI 上有的特性目前都不支持 **Windows**(如 服务发现, 健康检查, 元数据, DNS, 负载均衡)。

> **注意:** Rancher 有一个默认的 Windows 环境模版。如果你想在Windows中创建一个环境，你需要禁用所有其它的基础架构服务，因为这些服务目前都不兼容Windows。

### 创建一个 Windows 环境
在环境下拉框中点击 **环境管理**，进入环境管理页面后点击**添加环境**，填写**名称**, **描述** (可选)，选择一个有Windows编排的环境模版。如果启用了[访问控制]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control/)，你可以[添加成员]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/#editing-members)，设置他们的[成员角色]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/#membership-roles)。任何被加入环境成员列表的用户都能访问这个环境。

创建一个 Windows 环境后，你可以通过选择左上角的环境下拉框中的环境名， 或在特定环境下拉框中选择 **切换到此环境** 的方式进入到这个环境

> **注意:** 因为 Rancher 支持多个容器编排框架， Rancher 目前不支持在已经有服务运行的环境中切换编排框架。

### 添加 Windows 主机
要在 Windows Rancher 中 添加一个主机，你需要先有一个运行了 [Windows Server 2016 with Docker](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/about/index)的主机。

在 **基础架构** 板面，你可以用自定义命令启动 Rancher 客户端服务。你可以按照指示在Windows启动 Rancher 客户端服务。

Rancher 二进制客户端会被下载到 `C:/Program Files/rancher` 目录，日志客户端会被下载到 `C:/ProgramData/rancher/agent.log`。

### 移除 Windows 主机
作为添加一个主机到 Rancher 的一部分，Rancher 客户端在主机上被安装和注册为一个服务。你必须删除已经存在的服务，你可以在 powershell 中运行如下命令来删除服务。删除服务后你可以在 Windows 环境中重用这个主机

```bash
& 'C:\Program Files\rancher\agent.exe' -unregister-service
```

### Windows 中的网络
我们默认支持 NAT 和 [透明网络(transparent networking)](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/container-networking).

目前，默认的 **Windows** 环境模版支持名为transparent 的透明网络(transparent networking)
这个透明网络是在运行 `docker network create -d transparent transparent`时创建的。

如果你要创建一个名字不是 `transparent` 的透明网络(transparent networking)，你需要创建一个新的环境模版，并把 Windows 设为容器编排平台。选择**Windows**后，你可以点击 **编辑配置** 来更改透明网络的名字。你可以用这个环境模版创建一个环境。但在 Rancher UI 中这个透明网络的默认名字依然是 `transparent`。 因此，你需要把命令更新为 `docker network create -d transparent <NEW_NAME_IN_TEMPLATE`.
