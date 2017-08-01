---
title: Swarm in Rancher
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---

## Swarm
---

在Rancher中部署Swarm，你首先需要添加一个新的[environment]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/)，这个环境需要[environment template]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/#what-is-an-environment-template)中设定容器编排引擎是 **Swarm**。

### 需求
* Docker 1.13 以上
* 端口 `2377` 以及 `2378` 需要能在Host间互访问。

### 创建一个 Swarm 环境

在环境的下拉菜单中，点击 **环境管理**。通过点击 **添加环境** 去创建一个新的环境，需要填写 **名称**，**描述**（可选），并选择Swarm作为编排引擎的环境模版。如果启用了[access control]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control/)，你可以在环境中[add members]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/#editing-members)并选择他们的[membership role]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/#membership-roles)。所有被添加到成员列表的用户都能访问你的环境。

在创建Swarm环境后，你可以在左上角环境的下拉菜单中切换到你的环境，或者在环境管理页面中，在对应环境的下拉选项中点击 **切换到此环境** 。

> **Note:** 由于Rancher支持多种的容器编排引擎框架，Rancher目前不支持在多个已经有服务运行的环境之间切换容器编排引擎。

### 启动 Swarm

在Swarm环境呗创建后，[infrastructure services]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/)在你添加一台主机到这个环境之前是不会启动的。**Swarm** 服务会需要至少3个Host节点。[adding hosts]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/)的过程与其他类型的环境相同。一旦第一个Host被添加，Rancher将会自动启动基础架构服务，包括Swarm的组件（例如swarm与swarm-agent）。你可以在 **Swarm** 标签页看到部署的过程。

> **Note:** 请不要手工对docker daemon使用 `docker swarm` 命令。这可能与Rancher管理Swarm集群产生冲突。

> **Note:** 只有Rancher的管理员或者环境的所有者才能够看到[infrastructure services]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/)。

#### 在Shell中使用CLI

Rancher提供一种便利的Shell工具去访问集群，这个命令可以被用来执行 `docker` 或者 `docker-compose` 命令。
