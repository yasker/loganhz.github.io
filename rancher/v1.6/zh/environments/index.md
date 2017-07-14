---
title: Environments in Rancher
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---

## 环境 (Environments)
---

### 什么是一个 `环境`?
Rancher 支持将资源分组归属到多个 Environment。每个 Environment 具有自己独立的基础架构资源及服务，并由一个或多个用户、团队或组织管理。例如，您可以创建独立的“开发”、“测试”及“生产” Environment 以确保环境之间的安全隔离，将“开发” Environment 的访问权限赋予全部人员，但限制“生产” Environment 的访问权限给一个小的团队。

所有主机和 Rahcner 资源, 比如 容器, 基础架构服务等, 都在 Environment 创建, 并且属于一个 Environment。

### 添加 Environment

要添加一个 Environments，把鼠标移动到位于左上角的当前 Environment， 此时会出现一个带有所有可用的 Ｅnvironment 下拉框，以及一个 **环境管理** 连接。点击 **环境管理**。


导航到 **环境** 页面后，你会看到一个环境列表和一个环境模板列表。如果你是 Rancher 的
[admin]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/accounts/#admin) 用户，你会看到一个所有人的环境列表，即使你不是该环境的[一部分]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/#membership-roles)。对于所有用户， 所有环境模板都可用。

点击 **添加环境**。每个环境都有自己的名字和描述，你可以选择你要使用的环境模板。在环境模板中，你可以看到那个基础架构服务是启用的。

> **注意:** 如果没有配置 [访问控制]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control/)， 所有环境都可以被其它 Rancher 的用户访问到。 环境没有任何所属关系。

有两种方法可以添加成员到一个 环境:

-  提供用户名，点击 **+** 把用户添加到成员列表中。如果该用户名不在列表中，则不会被加入到环境中。
-  右侧有一个 **+** 下拉框按钮，在某些授权类型下，这个下拉框会出现组织/团队。


每个成员 (既 个人、团队、或组织)，你可以把自己定义成 [所有者(owner)](#owners)、 [成员(member)](#members)、 [受限制的成员( restricted user)](#restricted) 或 [只读用户(read only user)](#read-only) 中的一个角色。默认，这些角色都已经添加到了环境成员列表中。通过用户名旁边的下拉框，可以改变相应用户的角色. 对于owner 用户，你可以随时编辑成员列表以及成员角色。 只有环境的 owner 能编辑环境的成员以及其角色。

> **注意:** 只有 owner 和 admin 才能查看环境的 [基础架构服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/)。

点击 **创建** 会创建一个环境，所有在成员列表中的用户都立即可以看到这个环境. 创建完环境添加主机后，已启用的基础架构服务会开始部署。

### 停用和删除环境

创建环境后，所有者可能想停用或删除该环境。

环境被停用后，这个环境不再对环境成员可见，但环境的所有者还可以看到并启用这个环境。在环境停用后你不能变更环境的成员，直到该环境被再次启用。环境被停用后所有东西不能在变更，如果你要变更你的基础架构服务，你需要在环境停用之前变更。


要删除一个环境，先要停用这个环境。环境删除后，这个环境所有的注册信息，分支，API keys 都会从 Rancher 移除。所有通过 Rancher UI 创建，用 Docker Machine 启用的主机也会在云提供商中被用 Docker Machine 移除。如果你已经通过[自定义]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/custom/)的方式添加了一台主机，那么这台主机在云提供商中不会被移除.

### 成员编辑
只有环境的所有者可以编辑环境的成员。即使一个环境处于停用的状态，所有者也可以编辑该环境的成员。在**环境管理**页面，你可以点击**编辑**进入环境成员编辑页面， 在编辑页面，你可以通过下拉框添加环境成员。

如果要删除环境成员，可以点击成员列表旁边的**X**。注意，单个成员被删除时，如果被删除的成员所属的团队或组织是这个环境的成员，那么他们仍然可以访问这个环境，

所有者可以更改任何环境成员的角色，你只需要选择成员的相应角色。

### 成员角色

#### 所有者
所有者有改变成员和环境之间关系的权限。在环境的成员列表中，所有者还可以改变环境成员的角色。

由于无法环境模版编辑，所有者有通过[catalog]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/catalog)改变环境的[基础架构服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/)的权限。环境模版只有在环境环境时会被使用到。

#### 成员
一个环境的成员可以在Rancher里面做任何不影响环境本身的操作。成员不能添加／移除其他成员，不能改变其他已存在成员的角色，也不能查看任何[基础架构服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/)。

#### 受限

环境的受限成员只能够做与 应用(stacks)，服务(service)相关的操作。受限成员能能对于任意服务的容器做任何操作，即，启动、停止、删除、升级、克隆和编辑。从应用、 服务、和容器操作的角度来说，受限成员是不受限制的。

对受限成员的限制体现在他们对主机的操作上。受限成员只能查看一个环境的主机，而不能添加，编辑，移除环境的主机。

> **注意:** 受限成员不能添加、移除主机标签，只有成员所有着才能改变主机的标签。


#### 只读
只读成员只能查看环境的资源。 他们可以查看 主机、应用、服务和容器。但只读成员不能对它们作任何创建、编辑、移除操作。


> **注意:** 只读成员可以查看容器的日志。

### 什么是环境模版

环境模版可以让用户定义需要部署的基础架构服务组合。基础架构服务包括（但不限于）容器编排 (即 cattle，[kubernetes]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/kubernetes/)、[mesos]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/mesos/)、[swarm]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/swarm/))、[网络]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/networking/)、rancher 服务 (即 [健康检查]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/health-checks)、[dns]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/dns-service/)、[metadata]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/metadata/)、[调度]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/scheduling/)、服务发现、[存储]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/storage-service/)。

容器的编排方式很多，Rancher 提供了一套默认的模版，以及推荐使用的基础架构服务用于容器编排。其中的一些基础架构服务，如 Rancher 调度器只能在cattle环境下使用，但其编排方式依赖这些基础架构服务。因为这些服务被用来启动其它基础架构服务。除了默认的模版，你也可以创建自己的模版。通过自己创建模版，你可以选者环境中任何你想要的基础架构服务组合。只有 [所有者](#owners) 或 [管理员]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control/#admin) 可以查看和编辑和管理环境的基础架构资源。

在和其它用户共享环境前， 我们推荐先设置好[权限控制]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control/)。用户被加入一个环境后, 他们就拥有了创建服务，和管理资源的权限。

> **注意:** 基础架构资源不可以夸环境共享. [注册]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/registries/)、[证书]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/certificates/) 和 环境 [API keys]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/api/api-keys/) 也不能夸环境.

### 添加环境模版

要添加一个新环境，你可以把鼠标移动到左上角的环境下拉框。下拉框中会出现所有可用的环境以及**环境管理**的链接。 点击**环境管理**

在**环境**页面后，你可以看到一个环境列表和一个环境模版列表。 点击**添加模版**

为模版选择一个 **名称** 和 **描述**， 选择分享自己模版的方式。 模版可以是私有（只有自己可见）和公有（管理员可见）。

[基础架构服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/) 包括，但不限于容器编排、[存储]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/storage-service/)和[网络]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/networking)。默认的基础架构服务已自动启动，它要求有一个可用的环境。

### 编辑 & 删除环境模版
创建环境模版后，你可以在模版中编辑启用哪个基础架构服务。虽然环境模版是可以编辑的，但已经存在的基于模版创建的环境不会随模版自动更新。

你可以在任何时候删除一个环境模版，因为它们只有在启动环境的时候才会被用到（用来指定哪些基础架构服务会被启用）。环境与环境模版没有直接绑定关系，所以删除环境模版不会影响环境。

