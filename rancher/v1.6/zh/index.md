---
title: Rancher概览
layout: rancher-default-v1.6
version: v1.6
lang: zh
redirect_from:
  - /rancher/latest/zh/
  - /
  - /rancher/
  - /rancher/latest/
  - /rancher/latest/en/
---

## Rancher概览
---

Rancher是一个开源的可用于生产环境的企业级容器管理平台。通过Rancher，企业不必利用一系列的开源软件去从头搭建自己的容器服务平台。Rancher提供了在生产环境中使用的全栈化容器部署与管理平台。

Rancher由以下四个部分组成:

### 基础设施编排

Rancher takes in raw computing resources from any public or private cloud in the form of Linux [hosts]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/). Each Linux host can be a virtual machine or physical machine. Rancher does not expect more from each host than CPU, memory, local disk storage, and network connectivity. From Rancher’s perspective, a VM instance from a cloud provider and a bare metal server hosted at a colo facility are indistinguishable.

Rancher implements a portable layer of [infrastructure services]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/) designed specifically to power containerized applications. Rancher infrastructure services include [networking]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/networking), [storage]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/storage-service/), [load balancer]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/load-balancer/), [DNS]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/dns-service/), and security. Rancher infrastructure services are typically deployed as containers themselves, so that the same Rancher infrastructure service can run on any Linux hosts from any cloud.

### 容器编排与调度

Many users choose to run containerized applications using a container orchestration and scheduling framework. Rancher includes a distribution of all popular container orchestration and scheduling frameworks today, including [Docker Swarm]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/swarm), [Kubernetes]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/kubernetes), and [Mesos]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/mesos/). The same user can create multiple Swarm or Kubernetes clusters. They can then use the native Swarm or Kubernetes tools to manage their applications.

In addition to Swarm, Kubernetes, and Mesos, Rancher supports its own container orchestration and scheduling framework called Cattle. Cattle was originally designed as an extension to Docker Swarm. As Docker Swarm continues to develop, Cattle and Swarm started to diverge. Rancher will therefore support Cattle and Swarm as separate frameworks going forward. Cattle is used extensively by Rancher itself to orchestrate infrastructure services as well as setting up, managing, and upgrading Swarm, Kubernetes, and Mesos clusters.

### 应用商店

Rancher users can deploy an entire multi-container clustered application from the application [catalog]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/catalog) with one click of a button. Users can manage the deployed applications and perform fully automated upgrades when new versions of the application become available. Rancher maintains a public catalog consisting of popular applications contributed by the Rancher community. Rancher users can [create their own private catalogs]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/catalog/private-catalog/).

### 企业级权限管理

Rancher支持灵活的插件式的用户认证。支持Active Directory，LDAP， Github等 [认证方式]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control/). Rancher支持在[环境]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/)级别的基于角色的权限控制 (RBAC)，可以通过角色来配置某个用户或者用户组对开发环境或者生产环境的访问权限。

下图展示了Rancher的主要组件和功能

<img src="{{site.baseurl}}/img/rancher/rancher_overview_2.png" width="800" alt="Rancher Overview">
