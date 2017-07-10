---
title: Infrastructure Services in Rancher
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
redirect_from:
  - /rancher/latest/zh/rancher-services/
---

## 基础架构服务
---

当启动Rancher时，每一个[environment]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/)都是基于[environment template]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/#what-is-an-environment-template) 并且在environment template中，在启动一个环境时，你可以选在需要启动的基础架构服务。这些基础架构服务包括编排引擎，[external DNS]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/external-dns-service/)，[networking]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/networking/)，[storage]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/storage-service/)，框架服务 (i.e. [internal dns]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/dns-service/)，[metadata]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/metadata-service)，和[health check]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/health-checks))。

基础架构服务基于[Rancher catalog](https://github.com/rancher/rancher-catalog)和[community catalog](https://github.com/rancher/community-catalog)中的`infra-templates`文件夹中的模版。Rancher catalog和community catalog是默认开启的，它们提供了一系列可以在环境模版中使用的基础服务

当创建一个环境模版时，默认开启运行一个环境所需的一系列基础服务。
