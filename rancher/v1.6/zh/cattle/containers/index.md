---
title: Containers
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---

## 容器
---

### 添加容器

通常，我们建议人们使用[services（服务）]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/applications/stacks/adding-services) 添加容器，因为它为用户提供了一些更大的灵活性，但有时我们理解您可能需要升级一个容器。

在**Infrastructure（基础设施）** -> **Container（容器）**页面中，单击**Add Container（添加容器）**。任何docker run 支持的参数，同样的Rancher也支持。

1. 提供**Name（名称）**，如果需要，也可提供容器的**Description（描述）**。
2. 提供**Image（镜像）**使用。您可以使用DockerHub上的任何镜像以及已添加到Rancher的[registries]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/registries) 。镜像名称的语法与任何`docker run`命令相匹配。


    镜像名称的语法。默认情况下，我们从docker register中拉取。如果没有指定标签，我们将拉取标签为tag的镜像。

    `[registry-name]/[namespace]/[imagename]:[version]`

    <a id="port-mapping"></a>


3. 如果需要，设置端口映射，这提供了通过主机IP访问容器上暴露端口的方法。在**Port Map（端口映射）** 部分中，你可以定义用于与容器进行通信的公共端口。您还可以定义用于连接到容器的暴露端口的私有端口。

    **随机端口映射**

    如果您想要利用Rancher的随机端口映射，公共端口可以留空，您只需要定义私有端口。私有端口通常是容器上的暴露端口之一。



    > **注:** 当在Rancher中显示端口时，它只会在创建时显示端口。如果端口映射有任何编辑，它不会在`docker ps`中更新，因为Rancher通过管理iptable规则，使端口完全动态。

4. 在各种选项卡中，Docker中可用的所有选项均可用于Rancher。默认情况下，我们设置了`-i -t`。

    如果您选择从**Infrastructure(基础设施）** -> **Containers（容器）**页面添加容器，则Rancher会自动为您选择一个主机。否则，如果您选择了一个主机来添加容器，主机将被安装在**Security/Host**选项卡中。

    你还可以向容器添加标签以及应用调度规则。有关标签和调度的更多细节参考[这里]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/scheduling/)。
5. 当您的填写容器选项完成后，单击**Create（创建）**。


## 编辑容器
---

从容器的下拉列表中，您可以选择不同的操作来在容器上执行。在主机或服务器上查看容器时，可以通过将鼠标悬停在容器名称上来查找下拉图标。在**Infrastructure（基础设施）** -> **Containers（容器）**中，仅在主机上创建的容器才可以看到下拉图标。通过服务创建的任何容器都不会显示其下拉图标。

您可以随时点击容器名称，这将带您进入容器详细信息页面。在该页面上，下拉菜单位于容器状态旁边的右上角。

当您从下拉菜单中选择**Edit（编辑）**时，您只能更改容器的名称和说明。Docker容器在创建后是不可变的。唯一可以编辑的是我们存储的东西，但这些不是Docker容器的一部分。这包括重新启动，即使停止并启动它，它仍然是同一个容器。您将需要删除并重新创建一个容器来更改任何其他内容。

> **注:** 当Rancher暴露端口时，它不会出现在`docker ps`中，因为Rancher管理iptable规则以使端口完全动态。

您可以**克隆**，这将预先填充**添加容器**屏幕与现有容器的所有设置。如果您忘记了某些东西，您可以克隆容器，更改它，然后删除旧的容器。

### Changing the Container States

When a container is in a **Running** state, you can **Stop** the container. This will stop the container on the host, but will not remove it. After the container is in the _Stopped_ state, you can select **Start** to have the container start running again. Another option is to **Restart** the container, which will stop and start the container in one step.

You can **Delete** a container and have it removed from the host.

当容器处于**Running(运行)**状态时，你可以**Stop(停止)**容器。这将停止主机上的容器，但不会将其移除。容器处于`暂停`状态后，可以选择**Start(开始)**以使容器再次开始运行。另一个选项是**Restart(重新启动)**容器，该容器将在一个步骤中停止并启动容器。

您可以使用**Delete(删除)**使容器从主机中删除。


### 执行Shell

When you select **Execute Shell**, it brings you into the container shell prompt.
当您选择**执行Shell**，它将带您进入容器shell提示符。

### 查看日志

It's always useful to see what is going on with the logs of a container. Clicking **View Logs** provides the equivalent of `docker logs <CONTAINER_ID>` on the host.
通过查看日志去查看容器发生的情况是非常有帮助的。单击**View Logs（查看日志）**相当于docker logs <CONTAINER_ID>在主机上。
