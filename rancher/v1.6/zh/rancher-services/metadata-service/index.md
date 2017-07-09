---
title: Metadata Service in Rancher
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
redirect_from:
  - /rancher/rancher-services/metadata-service/
---

## Metadata 服务
---


Rancher通过基础架构中的metadata服务为你的services和contianer提供数据。 可以通过直接调用基于HTTP的API的形式访问这些数据，来管理你运行中的docker实例。 这些数据包括创建docker contianers，Rancher services的静态数据，或者运行时数据，例如：在同一个服务里的discovery information about peer containers. TODO

通过Rancher的metadata服务，你可以操作任何使用Rancher托管网络的容器，并且在Rancher中取回容器的信息。通过metadata可以获取container，service，包含container的stack，container所在的主机。metadata是JSON格式的。

有多重方式运行Rancher托管的containers。Rancher网络的原理详见[networking]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/networking)。

### 如何获取Metadata

通过Rancher UI，你可以通过container的下拉菜单 **执行命令行** 进入运行命令界面。The drop down can be found by hovering over the container。TODO

你可以通过curl命令获取metadata信息。

```bash
# If curl is not installed, install it
$ apt-get install curl
# Basic curl command to obtain a plaintext response
$ curl http://rancher-metadata/<version>/<path>
```
curl请求的path取决于你想要获取的metadata信息和格式。

Metadata | path  | 描述
----|---- | ---
Container | `self/container` | 提供运行命令的container的metadata信息
Service that container is part of | `self/service` | 提供运行命令的container对应service的metadata信息
Stack that container is part of | `self/stack` | 提供运行命令的container对应stack的metadata信息
Host that container is deployed on | `self/host` | 提供运行命令的container对应host的metadata信息
Other Containers | `containers` | 提供所有containers的metadata信息。在plaintext格式，提供了带上索引序号的所有容器。在JSON格式，提供了所有containers的所有metadata信息。使用序号或者名字。都可以获取指定container的metadata信息。
Other Services | `services` | 提供了所有services的metadata信息。在plaintext格式，提供了带上索引序号的所有services。在JSON格式，提供了所有services的所有metadata信息。在路径中使用序号或者名字，都可以获取指定service的metadata信息。当访问container详细信息时，在V1 (`2015-07-25`)只返回container name(s)，但是在V2 (`2015-12-19`)，container object(s) 也会返回。
Other Stacks | `stacks/<stack-name>` | 提供了所有stack的metadata信息。在plaintext格式，提供了带上索引序号的所有stack。在JSON格式，提供了所有stack的所有metadata信息。使用序号或者名字。都可以获取指定容器的metadata信息。在路径中使用序号或者名字，都可以获取指定stack的metadata信息。当访问container详细信息时，在V1 (`2015-07-25`)只返回container name(s)，但是在V2 (`2015-12-19`)，container object(s) 也会返回。

### Metadata的版本

在`curl`命令中，我们强烈建议使用确定的版本号，但是你也可以选者`latest`。

> **注意:** 因为`latest`版本会包含最新的代码变动，各个版本的返回的数据可能不同，需要确认是否和你之前的代码能够兼容。

metadata的版本是基于日期的。

Version Reference | Version|
---- | ----
V2 | 2015-12-19 |
V1 | 2015-07-25 |

#### 版本变化

##### V1 vs. V2

当通过http请求访问路径 `/services/<service-name>/containers`或者`/stacks/<stack-name>/services/<service-name>/containers`时, V1 返回container name(s)，V2返回container object(s)。更多详细信息在V2 metadata服务中提供。

##### 范例

在Rancher中，名为`foostack`的stack包含一个有三个容器的service `barservice`。

```bash
# 在V1只返回service的container names
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-07-25/services/barservice/containers'
["foostack_barservice_1", "foostack_barservice_2", "foostack_barservice_1"]

# 在V2中返回service的container objects
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-12-19/services/barservice/containers'
[{"create_index":1, "health_state":null,"host_uuid":...
...
# 获取service中所有容器的metadata信息
...
...}]

# 在V2，可以获取指定的container object
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-12-19/services/barservice/containers/foostack_barservice_1'
[{"create_index":1, "health_state":null,"host_uuid":...
...
# 获取service中所有容器的metadata信息
...
...}]

# 通过路径 /stacks/<service-name>，可以访问services和containers

# 使用V1只返回service的container names
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-07-25/stacks/foostack/services/barservice/containers'
["foostack_barservice_1", "foostack_barservice_2", "foostack_barservice_1"]

# 使用V2返回service的container objects
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-12-19/stacks/foostack/services/barservice/containers'
[{"create_index":1, "health_state":null,"host_uuid":...
...
# 获取service中所有容器的metadata信息
...
...}]
```

### Plaintext vs JSON

Metadata返回有plaintext和JSON两种格式，根据需要选择相应格式.

#### Plaintext

通过curl命令，会获得请求路径的plaintext格式返回。你可以通过从第一层路径开始，层层推进，找到你需要的信息。

```bash
$ curl 'http://rancher-metadata/2015-12-19/self/container'
create_index
dns/
dns_search/
external_id
health_check_hosts/
health_state
host_uuid
hostname
ips/
labels/
memory_reservation
milli_cpu_reservation
name
network_from_container_uuid
network_uuid
ports/
primary_ip
primary_mac_address
service_index
service_name
stack_name
stack_uuid
start_count
state
system
uuid
$ curl 'http://rancher-metadata/2015-12-19/self/container/name'
# Note: Curl 不会返回新的行，只有一个数据时返回会输出在同一行
Default_Example_1$root@<container_id>
$ curl 'http://rancher-metadata/2015-12-19/self/container/label/io.rancher.stack.name'
Default$root@<container_id>
# Arrays可以通过序号或者名字访问
$ curl 'http://rancher-metadata/2015-12-19/services'
0=Example
# 使用序号或者名字
$ curl 'http://rancher-metadata/2015-12-19/services/0'
$ curl 'http://rancher-metadata/2015-12-19/services/Example'
```

#### JSON

JSON格式的返回可以通过在curl命令中增加header `Accept: application/json`

```bash
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-12-19/self/container'
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-12-19/self/stack'
# 获取stack中另一个service的信息
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-12-19/services/<service-name>'
```

### Metadata Fields

#### Container

| Fields | Description |
| ----| ----|
| `create_index` | container启动的序号 i.e. 2 代表的是service中启动的第二个container。Note: Create_index不会被重用。 如果你的service包含2 containers，删除了第二个container，下一个启动的container的`create_index`会是3，即使service中只包含2个container
| `dns` | container的DNS server。
| `dns_search` | container的搜索域。
| `external_id`  | 在host上的Docker container ID。
| `health_check_hosts` | 列出运行health checks的host的UUIDs。
| `health_state` | 开启health check的container的健康状态 [health check]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/health-checks/)。
| `host_uuid` | Rancher server分配给hosts的唯一标识。
| `hostname` | container的hostname。
| `ips` | 支持多NICs时的IP列表
| `labels` | [container 标签]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/scheduling/#labels)列表。格式为`key`:`value`。
| `memory_reservation` | container可以使用内存的软限制。
| `milli_cpu_reservation` | container可以使用CPU的软限制，值为正整数，1代表1/1000CPU。所以，1000 代表1个CPU，500代表半个CPU。
| `name` | container的名字。
| `network_from_container_uuid` | （TODO）container's UUID where the network is from.
| `network_uuid` | Rancher分配的network唯一标识
| `ports` | 列出 [contaienr使用的ports]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-services/#port-mapping)。格式为： `hostIP:publicIP:privateIP[/protocol]`.
| `primary_ip` | container IP
| `primary_mac_address` | container的primary MAC地址
| `service_index` | service中container name的最后一个数字
| `service_name` | service name(if applicable) （TODO）
| `stack_name` | service所在的stack的名字(if applicable) （TODO）
| `stack_uuid` | Rancher分配的stacks的唯一标识
| `start_count` | container启动的次数
| `state` | container状态
| `system` | container是否是Rancher基础架构[infrastructure service]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/)
| `uuid` | Rancher分配containers唯一标识

#### Service

 Fields | Description
----|----
`containers` | 列出service中的container names
`create_index` | container启动的序号 i.e. 2 代表的是service中启动的第二个container。Note: Create_index不会被重用。 如果你的service包含2 containers，删除了第二个container，下一个启动的container的`create_index`会是3，即使service中只包含2个container
`expose` | 对主机暴露，但是不对外暴露的端口
`external_ips` | [内部服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-external-services/) 的IP列表
`fqdn` | service的全称域名
`health_check` | 服务的 [健康检查配置]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/health-checks/)
`hostname` | [内部服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-external-services/)的CNAME
`kind` | Rancher Service类型
`labels` | [服务标签]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/scheduling/#labels)列表，格式为 `key:value`.
`lb_config` | [负载均衡]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-load-balancers/)的配置
`links` | 列出服务的链接，格式为`stack_name/service∂_name:service_alias`. `links`(i.e. `stack_name/service_name` 获取所有链接)根据返回的`service_alias`,获取进一步的详细信息。
`metadata` | [用户添加 metadata]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/metadata-service/#adding-user-metadata-to-a-service)
`name` | 服务名称
`ports` | [Service使用的端口]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-services/#port-mapping)。格式`hostIP:publicIP:privateIP[/protocol]`.
`primary_service_name` | 主service名，如果有sidekicks
`scale` | Service的scale
`sidekicks` | [sidekicks]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-services/#sidekick-services)服务的名称列表
`stack_name` | service所有的stack的名称
`stack_uuid` | Rancher分配的stacks唯一标识
`system` | 是否是[基础架构服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/)
`uuid` | Rancher分配的services唯一标识

#### Stack

Fields | Description
----|----
`environment_name` | stack所在的[Environment]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/) 的名字
`environment_uuid` | Rancher分配的stacks唯一标识
`name` | [Stack]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/stacks/)名字
`services` | Stack中的Services列表
`system` | stack是否是[基础架构服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/)
`uuid` | Unique stack identifier that Rancher assigns to stacks

#### Host

Fields | Description
----|----
`agent_ip` | Rancher Agent的IP，i.e. `CATTLE_AGENT_IP`环境变量值。
`hostname` | [Host]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/)的名字
`labels` | [Host Labels]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/#host-labels)列表。格式为`key:value`.
`local_storage_mb` | host的存储数值，单位为MB
`memory` | host的内存数值，单位为MB
`milli_cpu` | host的CPU. 数值为整数，1代表1/1000的cpu。所以，1000代表1 CPU.
`name` | [Host]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/)的名字
`uuid` | Rancher server分配的hosts唯一标识

### 为Service添加用户Metadata

Rancher支持为服务添加用户metadata。现在只支持通过[Rancher Compose]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/rancher-compose/) 添加，metadata是`rancher-compose.yml`的一部分。`metadata` key对应的部分，yaml会被转化成JSON格式在metadata-service中使用

#### Example `rancher-compose.yml`

```yaml
service:
  # Scale of service
  scale: 3
  # User added metadata
  metadata:
    example:
      name: hello
      value: world
    example2:
      foo: bar

```

服务启动后，可以在`.../self/service/metadata` or in `.../services/<service_id>/metadata`看到metadata使用metadata service


#### JSON查询

```bash
$ curl --header 'Accept: application/json' 'http://rancher-metadata/latest/self/service/metadata'
{"example":{"name":"hello","value":"world"},"example2":{"foo":"bar"}}

```

#### Plaintext查询

```bash
$ curl 'http://rancher-metadata/latest/self/service/metadata'
example/
$ curl 'http://rancher-metadata/latest/self/service/metadata/example'
name
value
$ curl 'http://rancher-metadata/latest/self/service/metadata/example/name'
# # Note: Curl 不会返回新的行，只有一个数据时返回会输出在同一行
hello$root@<container_id>
```
