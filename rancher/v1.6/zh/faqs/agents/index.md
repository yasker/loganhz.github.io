---
title: FAQS about Rancher Agents/Hosts
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---
## FAQs 关于　Rancher agent/host
---

### 我如何配置一个位于代理服务器后面的主机？

要支持代理服务器后面的主机，你需要配置 Docker 的代理服务。详细说明参考添加自定义主机。


### Rancher Agent 无法启动的原因是什么？

#### 添加 `--name rancher-agent`

如果你从 UI 中编辑`docker run .... rancher/agent...`命令添加`--name rancher-agent`选项，那么 Rancher Agent 将启动失败。Rancher agent 在初始运行时会启动 3 个不同容器，1 个是运行状态和2 个停止状态。Rancher agent要成功连接到 Rancher Server 必须要有两个名字为`rancher-agent` 和 `rancher-agent-state`的容器，第三个与 docker 名称冲突的容器会被移除。

#### 使用一个克隆的虚拟机。

如果你使用一个克隆的虚拟机并尝试注册它，它将不能工作并在 Rancher-agent 日志中产生`ERROR: Please re-register this agent.`.Rancher 的唯一 ID保存在`/var/lib/rancher/state`，和克隆的虚拟机是相同的所以无法重新注册。

解决方法是在克隆的VM上运行以下命令： `rm -rf /var/lib/rancher/state; docker rm -fv rancher-agent; docker rm -fv rancher-agent-state`, 完成后可重新注册。

<a id="agent-logs"></a>

### 我在哪里可以找到 Rancher agent 容器的详细日志?

从 v1.6.0起在Rancher代理上运行`docker logs`容器将提供一组与代理相关的所有日志。


###如果验证你的主机注册设置是否正确？
How to check your host registration is set correctly?

如果你正面临Rancher Agent 和 Rancher　Server 连接问题，请检查主机设置。当你第一次尝试在 UI 中添加主机，你需要设置主机注册的 URL，该URL用于建立从主机到Rancher API服务器的连接。这个URL必须可以从你的主机访问到。 去验证它要登录到主机并执行curl命令：


```
curl -i <Host Registration URL you set in UI>/v1
```

你应该得到一个json响应。 如果认证开启，响应代码应为401.如果认证未打开，则响应代码应为200。

> **注意:** 使用正常的HTTP连接和websocket连接（ws：//）。 如果此URL指向代理或负载平衡器，请确保它已配置为处理Websocket连接。
###我该怎么固定主机 IP 和更改它，如果主机的 IP 变了（因为重启），我改怎么办？


当agent连接到Rancher Server时，它会自动检测agent的IP。 有时，所选择的IP不是要使用的IP，或者选择了docker bridge IP
, 如. `172.17.x.x`.

或者，你有一个已经注册的主机，当主机重启后获得了一个新的IP,
Alternatively, if you've already registered a host and your host has a new IP after a reboot, 这个 IP 将会和你的主机在 Rancher UI 中的 IP 不匹配。

您可以重新配置“CATTLE_AGENT_IP”设置，并将主机IP设置为您想要的。

当主机IP地址不正确时，容器将无法访问管理网络。 要使主机和所有容器进入管理网络，只需编辑命令[添加自定义主机]通过将新的IP指定为环境变量。 在主机上运行编辑的命令。 不要停止或删除主机上的现有代理！

```bash
$ sudo docker run -d -e CATTLE_AGENT_IP=<NEW_HOST_IP> --privileged \
    -v /var/run/docker.sock:/var/run/docker.sock \
    rancher/agent:v0.8.2 http://SERVER_IP:8080/v1/scripts/xxxx
```

### 如果我的主机没有使用 Rancher而直接被删除会发生什么?

如果你的主机直接被删除,  Rancher Server 会一直显示该主机.
主机会处于`Reconnecting`状态，然后转到`Disconnected`状态，这样你才能够从 UI 中**删除**这些主机。

如果你有一些添加了健康检查功能的服务部署在`Disconnected`主机上，它们会重新调度到其他主机上。


### 为什么同一主机在UI中多次出现?

如果你使用 boot2docker添加主机，主机不能持久化`var / lib / rancher`，这是Rancher用来存储用于标识主机的必要信息。
