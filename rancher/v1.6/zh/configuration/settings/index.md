---
title: Settings in Rancher
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---

## 系统设置
---

在Rancher的**系统管理** -> **系统设置**页面，我们允许为不同区域的产品定制Rancher。

### 主机注册

在启动任何主机之前，您将被要求完成主机注册。此注册设置您的Rancher服务器将如何连接到您的主机。如果您已经设置了[访问控制]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control)，则不会提示您设置主机注册地址，因为Rancher假定您的URL将被访问。

该设置确定您的主机将用于连接到Rancher API的站点URL。默认情况下，Rancher选择用于访问UI的站点URL。如果您选择更改地址，请确保指定应用于连接到Rancher API的端口。如果您使用SSL配置Rancher，请确保将协议更改为`https`。此注册设置决定了[添加自定义主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/custom/)的命令。

如果为Rancher启用[访问控制]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control/)功能，则只有**管理员**才能更改主机注册地址。默认情况下，第一个**管理员**是配置访问控制策略时第一个使用Rancher验证的用户。如果访问控制策略仍未配置，则该站点的任何用户都可以更新主机注册地址。可以在**系统设置** -> **主机注册地址**选项卡中更新此选项。

### 应用商店

默认情况下，[应用商店]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/catalog/)有三类可使用的应用：

* [Rancher基础架构](https://github.com/rancher/infra-catalog)所有基础架构服务的模版。
* [Rancher官方认证](https://github.com/rancher/rancher-catalog)包括Rancher认证过的应用模版。
* [Rancher社区共享](https://github.com/rancher/community-catalog)包含社区共享的模版。

由于系统设置页面只有[管理员]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/accounts/#admin)有访问权限，因此只有管理员有权限添加私有的应用到Rancher。添加一个应用就是添加一个应用名称和git地址。正确的git地址的格式可以参考[这里](https://git-scm.com/docs/git-clone#_git_urls_a_id_urls_a)。无论什么时候添加应用，它都马上能被应用。

如果要添加自己创建的私有应用，那么git仓库必须设置成[特定格式]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/catalog/private-catalog)。


### 信息统计

默认情况下，Rancher要求你选择允许收集匿名统计信息。这些数据使我们更好的了解我们的用户群，帮助改进Rancher产品。点击[这里]({{site.baseurl}}/rancher/telemetry/)了解更多。
