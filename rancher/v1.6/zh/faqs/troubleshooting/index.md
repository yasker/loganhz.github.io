---
title: Troubleshooting FAQs about Rancher
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
redirect_from:
  - /rancher/v1.6/en/faqs/
  - /rancher/faqs/
  - /rancher/faqs/troubleshooting/
---

## 故障排错及常见问题
---

请阅读有关Rancher服务器和Rancher代理/主机的更多详细常见问题。

本节假设您能够成功启动Rancher服务器并添加主机。

### 服务/容器

#### 求助！为什么我只能编辑容器的名称？

Docker 容器在创建之后就不可更改了. 唯一可更改的内容是我们要存储的不属于 Docker 容器本身的那一部分数据. 无论是停止、启动或是重新启动，它始终在使用相同的容器. 如需改变任何内容都需要删除或重新创建一个容器.
你可以**克隆**，即是选择已存在的容器提前在**添加服务**界面中填入所有要设置的内容，如果你忘记填入某项内容，可以通过克隆来改变它之后删除旧的容器.


#### 如何在 Rancher 中关联容器/服务？How do linked containers/services work in Rancher?

在 Docker 中，关联容器（在 `docker run`中使用`--link`），之后在容器的`/etc/hosts`查看链接状态。在 Rancher 中，我们不需要更改容器的`/etc/hosts`文件，而是通过运行一个内部 DNS服务器来关联主机，DNS 服务器会反馈给我们正确的 IP。


#### 求助! 我不能通过 Rancher的界面打开 Shell 或查看日志.  Rancher 是如何去访问容器的 Shell和日志?

Agent 有可能会联接公共网络，通过 Agent 去访问容器 Shell 或 Logs 的请求是不可信的。而从 Rancher 服务器中发出的请求包括一个 JWT（JSON Web Token)，JWT 是由服务器签名并且可由 Agent识别出请求的确来自服务器，JWT其中一部分包括有效期限，有效期为 5 分钟，如果JWT被拦截，可防止它被长时间使用。但如果不使用SSL，这一点尤为重要。

如果你运行`docker logs -f (rancher-agent　名称或 ID）`，日志会显示令牌失效的日期，随后检查 Rancher 服务器主机和 Rancher 代理主机的日期/时间的一致性。

#### 在哪里可以看到我的服务日志?

在服务的详细页中，我们提供了一个服务日志的标签**日志**。在**日志**标签中，列出了和服务相关的所有事件，包括时间戳、API事件相关描述，这些日志将会保留 24 小时。

### 跨主机通信

如果容器运行在不同主机上，不能够ping 通彼此, 可能是由一些常见的问题引起的.

#### 如何检查跨主机通信是否正常?

在**应用**->**基础设施**中，检查 `healthcheck` 应用的状态。如果是active跨主机通信就是正常的。

手动测试，你可以进入任何一个容器中，去 ping 另一个容器的内部 IP。在主机页面中可能会隐藏掉基础设施的容器，如需查看点击“显示系统容器”的复选框。


#### Are the IPs of the hosts correct in the UI?

通常来讲，主机将提供给 Docker 网桥一个 IP 地址，而不是真实的 IP 地址，通常是`172.17.42.1`或以`172.17.x.x`开头的IP。如果是这种情况，在使用`docker run`命令时用正常的 IP 地址来指定`CATTLE_AGENT_IP`环境变量。

```bash
$ sudo docker run -d -e CATTLE_AGENT_IP=<HOST_IP> --privileged \
    -v /var/run/docker.sock:/var/run/docker.sock \
    rancher/agent:v0.8.2 http://SERVER_IP:8080/v1/scripts/xxxx
```

#### 在 Ubuntu 上运行容器时彼此间不能正常通行.

如果你的系统开启了`UFW`,请关闭`UFW`或更改`/etc/default/ufw`中的策略为：

```
DEFAULT_FORWARD_POLICY="ACCEPT"
```

<a id="subnet"></a>

#### Rancher 的默认子网（`10.42.0.0/16`）在我的网络环境中已经被使用或禁止使用，我应该怎么去更改这个子网？


Rancher overlay 网络默认使用的子网是`10.42.0.0/16`。如果这个子网已经被使用，你将需要更改Rancher网络中使用的默认子网，在 `Rancher－compose.yml`文件中的使用合适子网配置`default_network`保证你的网络基础服务。


要更改Rancher的IPsec或VXLAN网络驱动程序，您将需要具有更新的基础架构服务的环境模板。创建新环境模板或编辑现有环境模板时，可以通过单击**编辑**来配置网络基础结构服务的配置。在编辑页面中，选择**配置选项**　>　**子网**输入不同子网，点击**配置**。在任何新环境中将使用环境模板更新后的子网，编辑已经有的环境模板不会更改现在的环境中的子网。
这个实例是通过升级网络驱动的`rancher-compose.yml`文件去改变子网为`10.32.0.0/16`.
Here's an example of the updated network driver's `rancher-compose.yml` to change the subnet to `10.32.0.0/16`.

```yaml
ipsec:
  network_driver:
    name: Rancher IPsec
    default_network:
      name: ipsec
      host_ports: true
      subnets:
      # After the configuration option is updated, the default subnet address is updated
      - network_address: 10.32.0.0/16
      dns:
      - 169.254.169.250
      dns_search:
      - rancher.internal
    cni_config:
      '10-rancher.conf':
        name: rancher-cni-network
        type: rancher-bridge
        bridge: docker0
        # After the configuration option is updated, the default subnet address is updated
        bridgeSubnet: 10.32.0.0/16
        logToFile: /var/log/rancher-cni.log
        isDebugLevel: false
        isDefaultGateway: true
        hostNat: true
        hairpinMode: true
        mtu: 1500
        linkMTUOverhead: 98
        ipam:
          type: rancher-cni-ipam
          logToFile: /var/log/rancher-cni.log
          isDebugLevel: false
          routes:
          - dst: 169.254.169.250/32
```

> **注意:** 随着 Rancher 通过升级基础服务来更新子网，以前通过API更新子网的方法将不再适用。

### DNS

<a id="dns-config"></a>

### 如何查看我的 DNS 是否配置正确?

如果你想查看 Rancher　DNS 配置，点击**应用** > **基础服务**。点击`network-services`应用，选择`metadata`,在`metadata`中，找到容器名为`network-services-metadata-dns-X`，通过 UI 选择该容器的 **执行命令行**。

```bash
$ cat /etc/rancher-dns/answers.json
```


#### CentOS

##### 为什么我的容器无法连接到网络?

如果你在主机上运行一个容器（如：`docker run -it ubuntu`）该容器不能与互联网或其他主机通信，那可能是遇到了网络问题。

Centos 默认设置`/proc/sys/net/ipv4/ip_forward`为`0`，这从底层阻断了 Docker 所有网络。Docker将此值设置为`1`，但如果在CentOS上运行`service restart network`，则将其设置为`0`。


<a id="lb-config"></a>

### Load Balancer

#### 为什么我的负载均衡是 `Initializing`状态?

负载平衡器自动对其启用健康检查。 如果负载均衡器处于初始化状态，则很可能主机之间的跨主机通信不起作用。

#### 我如何看我的负载均衡器的配置?

如果要查看负载平衡器的配置，你需要用进入负载平衡器容器内部查找配置文件，你可以在页面选择负载平衡容器的**执行命令行**


```bash
$ cat /etc/haproxy/haproxy.cfg
```

该文件将提供负载平衡器的所有配置详细信息。

#### 我在哪能找到 HAproxy 的日志?

HAProxy的日志可以在负载平衡器容器内找到。 负载平衡器容器的`docker logs`只提供与负载平衡器相关的服务的详细信息，但不提供实际的HAProxy日志记录。

```
$ cat /var/log/haproxy
```

### 高可用

#### Rancher Compose Executor 和 Go-Machine-Service不断重启.

在高可用集群中，如果你正在使用代理服务器后，如果rancher-compose-executor和go-machine-service不断重启，请确保正在使用代理协议。

### 认证

<a id="manually-turn-off-github"></a>

####求助！我打开了访问控制但不能访问 Rancher 了，我该如何重置 Rancher 禁用访问控制？

如果你的身份验证出现问题（例如你的GitHub身份验证已损坏），则可能会将其锁定在Rancher中。 要重新获得对Rancher的访问权限，您需要关闭数据库中的Access Control。 为此，您需要访问运行Rancher Server的主机。

```bash
$ docker exec -it <rancher_server_container_ID> mysql
```

> **注意:** 这个 `<rancher_server_container_ID>`将是具有Rancher数据库的容器。 如果您升级并创建了一个Rancher数据容器，则需要使用Rancher数据容器的ID而不是Rancher服务器容器。


访问 Cattle数据库.

```bash
mysql> use cattle;
```

Review the `setting` table.

```bash
mysql> select * from setting;
```

更改 `api.security.enabled` 为 `false` 清除 `api.auth.provider.configured` 值 . 此更改将关闭访问控制，任何人都可以使用UI / API访问Rancher服务器。

```bash
mysql> update setting set value="false" where name="api.security.enabled";
mysql> update setting set value="" where name="api.auth.provider.configured";
```

确认在`setting`表中进行更改。

```bash
mysql> select * from setting;
```

可能需要约1分钟才能在用户界面中关闭身份验证，但你可以刷新网页和访问关闭访问控制的 Rancher。
