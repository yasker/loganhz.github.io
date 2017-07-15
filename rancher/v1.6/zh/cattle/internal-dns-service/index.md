---
title: Internal DNS Service in Cattle Environments
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---

## Rancher Cattle环境中的内部DNS服务
---
在Rancher中，我们拥有自己的内部DNS服务，允许同一个[cattle环境]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/)中的任何服务都可以解析环境中的任何其他服务.

堆栈中的所有服务都可以通过`<服务名称>`解析，并且不需要在服务之间设置服务链接。 创建服务时，您可以定义`服务链接`以将服务链接在一起。 对于任何不同堆栈的服务，您可以通过`<服务名称>.<堆栈名称>`而不是`<服务名称>`来解析。 如果您想以不同的名称解析服务，您可以设置服务链接，以便服务可以由服务别名解析.

### 通过链接设置服务别名

在UI中，[添加服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-services/#adding-services-in-the-ui)时，展开**服务链接**部分，选择服务，并提供别名.

如果您使用Rancher Compose[添加服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/services/#adding-services-with-rancher-compose)，`docker-compose.yml`将使用`links`或`external_links`指令.

```yaml
version: '2'
services:
  service1:
    image: wordpress
    # 如果其他服务在同一个堆栈中
    links:
    # <service_name>:<service_alias>
    - service2:mysql
    # 如果另一个服务是不同的堆栈
    external_links:
    # <stackname>/<service_name>:<service_alias>
    - Default/service3:mysql
```

### `从容器`和服务连接

在启动服务时，您可能需要指定只在同一台主机上一起启动服务。 具体的用例包括尝试使用另一个服务中的`卷来自`或`net`时。 当添加一个[从容器]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-services/#sidekick-services)时，这些服务可以通过他们的名字自动地相互解析。 我们目前不支持通过从容器中的links/external_links来创建服务别名。

当添加一个从容器时，总是有一个主服务和从容器。 它们一起被认为是单个启动配置。 此启动配置将作为一组容器部署到主机上，1个来自主服务器，另一个从每个从容器中定义。 在启动配置的任何服务中，您可以按其名称解析主服务和从容器。 对于启动配置之外的任何服务，主服务可以通过名称解析，但是从容器只能通过`<从容器名称>.<主服务名称>`来解析。

### 容器名称

所有容器都可以通过其名称来全局解析，因为每个服务的容器名称在每个环境中都是唯一的。 没有必要附加服务名称或堆栈名称。

#### 例子

##### 在同一堆栈中的pinging服务

如果你执行一个容器的shell，你可以通过服务名称ping同一堆栈中的其他服务.

在我们的例子中，有一个名为`stackA`的堆栈，有两个服务，`foo`和`bar`.

在执行`foo`服务中的一个容器之后，你可以ping通`bar`服务.

```bash
$ ping bar
PING bar.stacka.rancher.internal (10.42.x.x) 58(84) bytes of data.
64 bytes from 10.42.x.x: icmp_seq=1 ttl=62 time=1.63 ms
64 bytes from 10.42.x.x: icmp_seq=2 ttl=62 time=1.13 ms
64 bytes from 10.42.x.x: icmp_seq=3 ttl=62 time=1.07 ms
```

##### 在不同的堆栈中的Pinging服务

对于不同堆栈的服务，您可以使用`<服务名称>.<堆栈名称>`在不同的堆栈中ping服务.

在这个例子中，我们有一个名为`stackA`的堆栈，它包含一个名为`foo`的服务，我们有另一个名为`stackB`的堆栈，它包含一个名为`bar`的服务.

如果我们执行`foo`服务中的一个容器，你可以用`bar.stackb`来ping.

```bash
$ ping bar.stackb
PING bar.stackb (10.42.x.x) 56(84) bytes of data.
64 bytes from 10.42.x.x: icmp_seq=1 ttl=62 time=1.43 ms
64 bytes from 10.42.x.x: icmp_seq=2 ttl=62 time=1.15 ms
64 bytes from 10.42.x.x: icmp_seq=3 ttl=62 time=1.27 ms
```

##### 在从容器中的pinging服务

Depending on which service you ping from, you can reach a sidekick service by either `<sidekick_name>` or `<sidekick_name>.<primary_service_name>`.

取决于你从哪个服务ping，您可以通过`<从容器名称>`或`<从容器名称>.<主服务名称>`来访问从容器服务。

在我们的例子中，我们有一个名为`stackA`的堆栈，它包含一个名为`foo`的服务，它有一个从容器`bar`和一个名为`hello`的服务。 我们也有一个堆栈叫`stackB`，它包含一个服务`world`。

如果我们执行`foo`服务中的一个容器，你可以直接用`bar'命令ping它。

```bash
# 在`foo`服务中的一个容器中，`bar`是一个从容器。
$ ping bar
PING bar.foo.stacka.rancher.internal (10.42.x.x) 56(84) bytes of data.
64 bytes from 10.42.x.x: icmp_seq=1 ttl=64 time=0.060 ms
64 bytes from 10.42.x.x: icmp_seq=2 ttl=64 time=0.111 ms
64 bytes from 10.42.x.x: icmp_seq=3 ttl=64 time=0.114 ms
```

If we exec into one of the containers in the `hello` service, which is in the same stack, you can ping the `foo` service by `foo` and the `bar` sidekick service by `bar.foo`.

如果我们执行在同一个堆栈中的`hello`服务的一个容器，你可以通过`foo`来ping`foo`服务和`bar.foo`来ping`bar`服务.

```bash
# 在`hello`服务中的一个容器内部，这不是服务/从容器的一部分
# Ping主服务(i.e. foo)
$ ping foo
PING foo.stacka.rancher.internal (10.42.x.x) 56(84) bytes of data.
64 bytes from 10.42.x.x: icmp_seq=1 ttl=62 time=1.04 ms
64 bytes from 10.42.x.x: icmp_seq=2 ttl=62 time=1.40 ms
64 bytes from 10.42.x.x: icmp_seq=3 ttl=62 time=1.07 ms
# Ping从容器(i.e. bar)
$ ping bar.foo
PING bar.foo (10.42.x.x) 56(84) bytes of data.
64 bytes from 10.42.x.x: icmp_seq=1 ttl=62 time=1.01 ms
64 bytes from 10.42.x.x: icmp_seq=2 ttl=62 time=1.12 ms
64 bytes from 10.42.x.x: icmp_seq=3 ttl=62 time=1.05 ms
```

如果我们执行`world`服务中的一个容器，它是不同的堆栈，你可以通过`foo.stacka`来ping`foo`服务和`bar.foo.stacka`来ping从容器`bar`.

```bash
# Inside one of the containers in the `world` service, which is in a different stack
# 在`world`服务中的一个容器内，它们位于不同的堆栈中
# Ping另一个堆栈`stacka`中的主服务(i.e. foo)
$ ping foo.stacka
PING foo.stacka (10.42.x.x) 56(84) bytes of data.
64 bytes from 10.42.x.x: icmp_seq=1 ttl=62 time=1.13 ms
64 bytes from 10.42.x.x: icmp_seq=2 ttl=62 time=1.05 ms
64 bytes from 10.42.x.x: icmp_seq=3 ttl=62 time=1.29 ms
# Ping另一个堆栈`stacka`中的从容器(i.e. bar)
$ ping bar.foo.stacka
PING bar.foo.stacka (10.42.x.x) 56(84) bytes of data.
64 bytes from 10.42.x.x: icmp_seq=1 ttl=62 time=1.23 ms
64 bytes from 10.42.x.x: icmp_seq=2 ttl=62 time=1.00 ms
64 bytes from 10.42.x.x: icmp_seq=3 ttl=62 time=0.994 ms
```

##### Ping容器名称

从同一个环境下的任何一个容器中，无论它们是否在相同的堆栈或服务中，您都可以用容器名称来ping其他容器，

在我们的示例中，我们有一个名为`stackA`的堆栈，它包含一个名为`foo`的服务。 我们还有另一个堆栈叫`stackB`，它包含一个名为“bar”的服务。 容器的名称是`<stack_name>-<service_name>-<number>`。

如果我们执行`foo`服务中的一个容器，你可以通过ping来访问`bar`服务中的相关容器.

```bash
$ ping stackB-bar-1
PING stackB-bar-1.rancher.internal (10.42.x.x): 56 data bytes
64 bytes from 10.42.x.x: icmp_seq=1 ttl=62 time=1.994 ms
64 bytes from 10.42.x.x: icmp_seq=2 ttl=62 time=1.090 ms
64 bytes from 10.42.x.x: icmp_seq=3 ttl=62 time=1.100 ms
```
