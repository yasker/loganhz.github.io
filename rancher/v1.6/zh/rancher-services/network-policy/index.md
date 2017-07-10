---
title: Network Policy in Rancher
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---

## 网络策略
---

Rancher允许用户在[environment]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/)中配置网络策略。网络策略允许你在一个环境中定义特定的网络规则。所有的container默认可以互相通信，但是在需要加入你的container时或许会有限制。

### 启动Network Policy Manager

当配置[environment template]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/#what-is-an-environment-template)，你可以启动 **Network Policy Manage** 组件。

如果你已经有一个启动的Rancher环境，你可以从[Rancher catalog]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/catalog/)启动 **Network Policy Manager**

> **Note:** Network Policy Manager现在只能在使用 Cattle container 编排引擎的时候使用。Environment templates基于编排引擎确定哪些组件可用，Rancher支持几乎所有的编排引擎。

### 通过UI管理网络策略规则
网络策略规则可以在每个environment设置页面中配置。点击下拉列表中的 **Manage Environments**，然后在需要配置的enviroment右侧点击edit按钮

在界面上有四个选择，`Allow`允许网络通信，`Deny`限制网络通信

* **Between Linked Services:** 这个选项用来控制两个service中链接的container
* **Within Service**: 这个选项用来控制service内的container
* **Within Stack**: 这个选项用来控制相同stack中不同service
* **Everything Else**: 这个选项用来控制上面不包含的情况

通常的配置为在**Everything Else**选择 `Deny`，其他的都选择`Allow`。

> **Note:** 规则的顺序为从左至右


### 通过API管理网络策略规则

对于[network]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/api/v2-beta/resources/network/) 资源，`defaultPolicyAction`和`policy` 字段定义了container间通信的工作规则。`policy`字段是内容为[network policy rules]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/api/v2-beta/resources/networkPolicyRule/)的有序数组。通过Rancher's [API]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/api/v2-beta/)，可以配置环境的网络策略

#### 获取Network的API Endpoint

要配置网络策略，需要找到相应的**Network**资源。network是environment的一部分，找到网络的URL为:

```
http://<RANCHER_SERVER_IP>/v2-beta/projects/<PROJECT_ID>/networks/<NETWORK_ID>`
```

怎么查找需要配置的网络的URL:

1. 点击啊**API**打开**Advanced Options**。在 **Environment API Keys**，点击 **Endpoint (v2-beta)**.
  > **Note**: 在UI上是`Environment`，在API是`project`。
2. 在environment links中查找**networks**，点击链接。
3. 查询你环境中启动的networking driver的名字。例如：可能为 `ipsec`。点击network的 **self**
4. 在右边的**Operations**中，点击**Edit**，在`defaultPolicyAction`中，你可以修改默认的网络策略，同时在`policy`字段，你可以管理你的网络策略规则。


### 默认策略

默认所有容器间可以互相通信，在API中，你可以看到`defaultPolicyAction`被设置成`allow`。

可以通过修改`defaultPolicyAction`为`deny`来限制所有容器间的通信

### 网络策略规则

网络策略规则配置container可以和一系列特定的container通信

#### 有链接的services内的containers

假设: Service A链接service B.

开启service A和service B之间的通信:

```json
{
  "within": "linked",
  "action": "allow"
}
```
> **Note:** service B的container不会初始化一个链接到service A。

关闭service A和service B之间的通信:

```json
{
  "within": "linked",
  "action": "deny"
}
```

在environment内任一链接services的网络策略规则适用于所有有链接的services


#### 同一service中的containers

开通同一service内containers的通信:

```json
{
  "within": "service",
  "action": "allow"
}
```

关闭同一service内containers的通信:

```json
{
  "within": "service",
  "action": "deny"
}
```

#### 同一stack中的containers

开通同一stack内containers的通信:

```json
{
  "within": "stack",
  "action": "allow"
}
```

关闭同一stack内containers的通信:

```json
{
  "within": "stack",
  "action": "deny"
}
```

#### 基于labels的containers

通过labels开通containers间的通信:

```json
{
  "between": {
    "groupBy": "<KEY_OF_LABEL>"
  },
  "action": "allow"
}
```

通过labels关闭containers间的通信:

```json
{
  "between": {
    "groupBy": "<KEY_OF_LABEL>"
  },
  "action": "deny"
}
```

### Examples

#### Container隔离

environment内的容器都无法和彼此通信

* 设置`defaultActionPolicy`为`deny`.

#### Stack隔离

同一个stack中的containers可以彼此通信，但是不能和其他stack中的containers通信

* 设置`defaultActionPolicy`为`deny`.
* `policy`中添加如下规则:

```json
{
  "within": "stack",
  "action": "allow"
}
```

#### Label隔离

Containers包含匹配的labels可以通信，这个规则通过label去划分可以相互通信的containers

假设在environment我们有如下一系列的stacks

```yaml
stack_one:
  service_one:
    label: com.rancher.department = qa
  service_two:
    label: com.rancher.department = engineering
  service_three:
    label: com.rancher.location = cupertino

stack_two:
  service_one:
    label: com.rancher.department = qa
  service_two:
    label: com.rancher.location = cupertino

stack_three:
  service_one:
    label: com.rancher.department = engineering
  service_two:
    label: com.rancher.location = phoenix
```

包含`com.rancher.department` label的containers可以相互通信

* 设置`defaultActionPolicy`为`deny`.
* 在`policy`中添加如下规则:

```json
{
  "between": {
    "groupBy": "com.rancher.department"
  },
  "action": "allow"
}
```

上面有两个不同的lable key pair(i.e. `com.rancher.department`)。

* Containers包含 `com.rancher.department = engineering`  彼此间可以通信，但是和其他的containers不能通信。在上面例子中，任何 `stack_one.service_two` 中的containers和 `stack_three.service_one`中的containers可以彼此通信，但是其他的不能。
* Containers包含 `com.rancher.department = qa`彼此间可以通信，但是和其他的不能。在上面的例子中，任何`stack_one.service_two`中的containers可以和任何`stack_two.service_two`中的containers通信，但是其他的不能。
* Containers不包含key `com.rancher.department`不能和其他containers通信
