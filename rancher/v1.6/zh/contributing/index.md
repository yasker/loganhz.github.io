---
title: Contributing to Rancher
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
redirect_from:
  - /rancher/latest/en/contributing/
---

## 支持 Rancher
---

### 开发

我们Githb的 [repository](https://github.com/rancher/rancher) wiki里有所有能帮助你参与Rancher开发的的步骤。

从我们的第一个[cowpoke](https://github.com/rancher/rancher/wiki/Cowpoke-1:-Getting-Started-with-Rancher) 开始！

### 代码库

我们的所有代码都在我们的Github [主页](https://github.com/rancher)。 这些代码库有很多为 Rancher 所用，下面是其中Rancher用到的主要的代码库的描述。

[Rancher Repo](https://github.com/rancher/rancher)： Rancher 的主要代码库，它整合了很多其它的代码库.

[Cattle Repo](https://github.com/rancher/cattle)：Rancher 所开发的功能代码库。

[UI Repo](https://github.com/rancher/ui): Rancher UI 代码库。

[Rancher Catalog Repo](https://github.com/rancher/rancher-catalog): 这个代码库包含了大多数 Rancher [应用商店]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/catalog) [基础架构服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/) 模板。这些模板(在`infra-templates`目录中)，在 [环境(environments)]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/) 中被用做 [环境模版(environment templates)]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/#what-is-an-environment-template)的一部分。我们欢迎社区参与 Rancher 应用商店其它[负载均衡 provider]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/}}) 的开发。

[Community Catalog Repo](https://github.com/rancher/community-catalog)： 这个代码库包含了所有社区贡献的 [Rancher 应用商店]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/catalog)模板。我们欢迎社区参与 Rancher 应用商店模板的开发。

[Rancher CLI Repo](https://github.com/rancher/cli)： [Rancher 命令行]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cli/) 基于此代码库。

[Rancher Compose Repo](https://github.com/rancher/rancher-compose): [Rancher Compose]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/rancher-compose/) 命令行基于此代码库。 此代码库和 docker/libcompose 保持同步。

### Bugs

如果您发现 bug，或遇到任何问题，请在 github 上提 [issue](https://github.com/rancher/rancher/issues/new)。 虽然我们有很多 Rancher 相关的代码库，我们希望您把所有的 issue 提交到 [Rancher repo](https://github.com/rancher/rancher) 以免我们遗漏。

如果您跟新了 Rancher 的文档，请提交 PR 到我们的 [文档代码库](https://github.com/rancher/rancher.github.io) 或点击 **Edit this page** 直接跳转到编辑页面。
<br>
