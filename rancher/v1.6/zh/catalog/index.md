---
title: Rancher Catalog
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
redirect_from:
  - /rancher/catalog/
  - /rancher/latest/en/catalog/
---

## 应用商店
---

Rancher提供一个应用程序模版的应用商店，可以简化部署复杂Stack的过程。进入 **应用商店** 标签页，你可以看到所有的[enabled catalogs]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/settings/#catalog)的应用模版。在 **官方认证** 应用商店下包含了[Rancher certified catalog](https://github.com/rancher/rancher-catalog) 中的应用模版，在 **社区共享** 应用商店下包含了[community-catalog](https://github.com/rancher/community-catalog) 中的应用模版。Rancher只会维护支持在官方认证的 _认证_ 模版。

### 添加应用商店 Catalogs

添加一个应用商店只需要添加一个应用商店名称，URL地址以及一个分支名称。URL地址需要 `git clone` [处理](https://git-scm.com/docs/git-clone#_git_urls_a_id_urls_a)。这分支名称需要是你应用商店git地址的一个分支。如果不提供分支名称，我们默认会使用 `master`。 无论你什么时候添加一个应用商店项目，他都会在你的应用商店中立刻生效。

目前有两类的应用商店可以添加到Rancher。全局应用商店以及环境应用商店。在全局的应用商店中，应用商店的模版会在所有的环境中。在环境应用商店中，应用商店的模版只会在对应的环境中生效。

Rancher的[admin]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control/#admin)可以在 **系统管理** -> **系统设置** 添加或者移除全局应用商店。

Rancher环境的用户可以在特定的Rancher环境中通过 **应用商店** -> **管理** 去添加或者删除环境应用商店。

如果你在一个代理后方运行Rancher server，你需要[start Rancher with certain environment variables]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/installing-rancher/installing-server/#http-proxy)是的Rancher应用商店正常运行。

### 应用商店中的基础架构服务

在[environment template]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/#what-is-an-environment-template)中启用生效的[infrastructure services]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/) 是在Rancher中生效的应用商店中的 `infra-templates` 文件夹。

应用架构服务同样在 **应用商店** 中可以哦那个，你可以看到所有（甚至与本环境不兼容的服务）的基础架构服务。建议在创建环境模版中选择基础架构服务而不是从应用商店中启用。

### 部署应用模版

Search for your desired template or use the filters for category or catalog. Once you have found your template, click on **Launch**. Fill in the required information for the template.
查询你需要的模版或者使用分类选择你需要的应用模版。当你完成选择你的template，点击 **启动**。填写模版需要信息。

1. 默认下使用最新 **版本** 的应用模版，不过需要的话，你也可以选择旧的版本。
2. 选择一个 **stack** 的名字，按需填写stack的 **描述**。
3. 填写配置选项，这些都是所选的应用模版需求的配置选项。
4. 点击 **创建** 去创建一个基于模版的stack。在创建stack之前，你可以通过点开 **预览** 窗口查看用于生成stack的 `docker-compose.yml` 和 `rancher-compose.yml`文件。

在你点击 **创建** 后，你的应用展会马上被创建，但没有启动任何服务。在stack的下拉菜单中点击 **启动服务** 去启动stack所有的容器。

### 更新应用模版

当一个新版本的应用模版被上传到catalog后，Rancher会提示你，这个应用有新版本可以更新。当你点击 **升级可以用**，你可以选择需要升级的版本。请在升级前仔细检查，以发现升级带来的潜在风险。在选择一个版本后，点击保存之前需要检查 **配置选项**。

在所有的服务都被更新后，stack和服务将会处于 **Upgraded** 状态。如果你确认已经升级成功，最后只需要在stack的下来菜单中点击 **完成升级**。**Note: 一旦完成升级，你将不能回退到旧版本。**

#### 回滚

如果升级过程中出现了问题，并且你需要回退到之前的版本。你可以在stack的下拉菜单中选择 **回滚**。
