---
title: Adding Azure Hosts
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---

## 添加Azure主机
---

Rancher支持使用`docker machine`部署[Microsoft Azure](https://azure.microsoft.com)。

### 启动Azure主机

1. 使用滚动条选择你要启动的主机的数量。
2. 为主机提供一个**名称**，如果需要的话写一点**描述**。
3. 提供Azure的**账户访问**信息，包括**用户名**，**密码**，**订阅ID**和**订阅证书**。

    > **注意：**如果你粘贴了你的订阅证书，请确保是一个base64字符串。

4. 选择你需要启动的**镜像**，只要`docker machine`在Azure上支持的镜像，Rancher都支持。
5. 选择镜像的**大小**。
6. 如需要，根据自己的情况更新**SSH端口**，**Docker端口**，**Docker Swarm主端口**。
7. Add your **Publish Settings File**.添加你的**发行设置文件**。
8. 选择你的Azure资源所在的**区域**。
9. (可选)向主机添加**[标签]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/#labels)**，以帮助组织主机并[调度服务/负载均衡器]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/scheduling/)或者是[使用除主机IP之外的其他IP解析外部DNS记录]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/external-dns-service/#using-a-specific-ip-for-external-dns).
10. (可选)在**高级选项**中，你可以利用[Docker引擎选项](https://docs.docker.com/machine/reference/create/#specifying-configuration-options-for-the-created-docker-engine)定制你的`docker-machine create`工具。
11. 所有的完成之后，点击**创建**。

一旦你点击创建，Rancher将会创建Azure虚拟机，并在实例中开启_rancher-agent_容器。几分钟之后，主机将会启动并可以[添加服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-services/)。
