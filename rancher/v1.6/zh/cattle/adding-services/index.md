---
title: Services
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---

## Services
---


* Cattle对服务采用标准Docker Compose术语，并将基本服务定义为从同一Docker映像创建的一个或多个容器。一旦服务（消费者）链接到同一个[stack（栈）]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/stacks/)中的另一个服务（生产者），映射到每个容器实例的[DNS 记录]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/internal-dns-service/) 就被容器从“消费”服务自动创建和发现。在Rancher创建服务的其他好处包括：



* 服务高可用性（HA）：Rancher不断监控服务中的容器状态，并主动管理以确保所需的服务规模。当健康的容器小于（甚至更小于）正常服务所需容器规模，主机不可使用，容器故障或者不能继续健康检查就会被触发。




* [healthcheck（健康监测）]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/health-checks/): Rancher通过`healthcheck`在其主机上运行基础架构服务来实施健康监控系统，以协调容器和服务的分布式健康检查。这些`healthcheck`容器内部使用HAProxy来验证应用程序的运行状况。当在单个容器或服务上启用健康检查时，将监控每个容器。



### 用户界面中的服务选项

在以下示例中，我们假设您已经创建了一个 [stack（栈）]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/stacks/), 设置了您的 [Hosts（主机）]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/), 并准备好构建应用程序。

我们将在添加服务时查看一些选项，最后将介绍如何创建一个链接到Mongo数据库的[LetsChat](http://sdelements.github.io/lets-chat/) application linked to a Mongo database.应用程序。


在`stack`中，您可以通过单击`Add Service（添加服务）`按钮添加服务。或者，如果要在`stack`级别查看 ，则对于每个单个`stack`，可以看到相同的`add service`按钮。




在**Scale（规模）**（规模）部分，您可以使用滑块来指定要为服务启动的特定数量的容器。或者，您可以选择**Always run one instance of this container on every host（始终在每个主机上运行此容器的一个实例）**。使用此选项，您的服务将扩展到添加到您的[环境]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/)中的任何其他主机。如果您在**Scheduling（调度）**选项卡中创建了调度规则，则Rancher将仅在符合调度规则的主机上启动容器。

您还需要提供**Name（名称）**，如果需要，还可以提供服务**Description（描述）**。



为服务提供所需的**Image（镜像）**。您可以使用[DockerHub](https://hub.docker.com/)上的任何镜像，以及已添加到您的[environment]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/)中的任何[registries]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/registries)。镜像名称的语法与任何`docker run`命令相匹配。


镜像名称的语法。默认情况下，我们从docker register中拉取。如果没有指定标签，我们将拉取标签为tag的镜像。

`[registry-name]/[namespace]/[imagename]:[version]`

在镜像名称下方，有一个复选框`Always pull image before creating`。默认情况下，选择这个选项。选择此选项后，在主机上启动容器的时候，服务的镜像将**始终**被拉到主机上，即使该镜像已被缓存。



#### 选项

Rancher努力与Docker保持一致，我们的目标是，支持任何`docker run`所支持的选项。端口映射和服务链接显示在主页面上，但所有其他选项都在不同的选项卡中。

默认情况下，服务中的所有容器都以分离模式运行，例如：`docker run`命令中的`-d`。


##### 端口映射



当我们映射端口时，我们创建了将容器的暴露端口访问主机上的公共端口的功能。在**Port Map端口映射** 部分中，你定义的公共端口将会暴露在主机上。该端口将流量指向定义的私有端口。私有端口通常是容器上暴露的端口（例如：`EXPOSE`在镜像的[Dockerfile](https://docs.docker.com/engine/reference/builder/#expose) 中）。当你映射一个端口时，Rancher将在尝试启动容器之前检查主机，以查看主机是否有端口冲突。



当使用端口映射时，如果服务的规模大于具有可用端口的主机数量，则您的服务将阻塞在正在激活状态。如果您查看服务的详细信息，您将可以看到一个`Error`状态的容器，这将表明容器由于无法在主机上找到打开的端口而失败。该服务将继续尝试，如果主机/端口可用，则该服务将在该主机上启动一个容器。



> **注意:** 当在Rancher中显示端口时，它只会在创建时显示端口。如果端口映射有任何编辑，它不会在`docker ps`中更新，因为Rancher通过管理iptable规则，使端口完全动态。

##### 随机端口映射

如果您想要利用Rancher的随机端口映射，公共端口可以留空，您只需要定义私有端口。



##### 链接服务


如果您的环境中已经创建了其他服务，则可以将已有服务链接到您正在创建的服务。链接服务将正在创建的服务中的所有容器，都链接到链接服务中的所有容器中。链接服务就像`docker run`命令中的`--link`功能一样。

Linking services is additional functionality on top of Rancher's [internal DNS]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/internal-dns-service/) and is not required to resolve services by service name.

链接服务是Rancher [internal DNS(内部DNS)]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/internal-dns-service/) 之上的附加功能，不需要按服务名称解析服务。

#### Rancher 选项

除了提供`docker run`支持的所有选项之外，Rancher还额外提供了UI相关概念。

##### HEALTH CHECKS（健康检查）

如果主机在Rancher中关闭（例如：处于`reconnecting`或`inactive`状态），您将需要执行健康检查，以使Rancher将服务上的容器启动到不同的主机。



> **注意:** `health check`仅适用于`managed`网络的服务。如果你选择任何其他网络，则**不能**被监察到。

在**Health Check**选项卡中，你可以选择检查服务的TCP连接或HTTP响应。

阅读有关Rancher如何处理[health checks]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/health-checks/)的更多详细信息。

##### 标签/调度


在**Lables（标签）**选项卡中，Rancher允许您将任何标签添加到服务中的容器中。标签在创建调度规则时非常有用。在**Scheduling（调度）**选项卡中，您可以使用[host labels（主机标签）]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/#host-labels)，容器/服务标签，和容器/服务名称来创建你服务需要调度的容器。

Read more details about [labels and scheduling]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/scheduling/).

阅读有关[labels and scheduling（标签与调度）]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/scheduling/)的更多细节。

### 在UI中添加服务



首先，我们通过设置`scale`为1的容器来创建我们的数据库服务，给它一个名称`database`，并使用`mongo:latest`镜像。没有使这个服务运行的其他选项需要设置了，因此单击**Create（创建）**。该服务立即开始启动。


现在我们已经启动了我们的数据库服务，我们将把web服务添加到我们的`stack`中。这一次，我们将服务规模设置为2个容器，提供另一个名称`web`并使用`sdelements/lets-chat`作为镜像。我们不会暴露Web服务中的任何端口，因为我们将计划对此服务进行负载均衡。由于我们已经创建了数据库服务，我们将在**Service Links（服务链接）**的`Destination Service(定义服务)`选择数据库服务，在`As name`中填写`mongo`。点击**Create（创建）**，我们的[LetsChat](http://sdelements.github.io/lets-chat/)应用程序已准备好使负载均衡服务指向它。

### Rancher Compose 中的服务选项

阅读更多关于[创建 Rancher Compose]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/rancher-compose/)的细节。


Rancher Compose工具就像受欢迎的Docker Compose一样工作，并支持V1和V2版本的docker-compose.yml文件。要启用Rancher支持的功能，您还可以使用rancher-compose.yml文档，它扩展和重写了docker-compose.yml。例如，rancher-compose.yml文档包含了服务的`scale`和`healthcheck`服务。


如果您不熟悉Docker Compose或Rancher Compose，我们建议您使用UI来启动您的服务。你可以通过单击`stack`的下拉列表中的**View Config（查看配置）**来查看整个stack的配置（例如：stack里面等效的docker-compose.yml文件和rancher-compose.yml文件）。

#### 链接服务

在Rancher中，环境中的所有服务都是DNS可解析的，因此不需要明确链接服务，除非您希望使用特定的别名进行DNS解析。

> **注意:** 我们目前不支持将sidekick服务与主服务相关联，反之亦然。阅读更多关于[Rancher内部DNS工作原理]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/internal-dns-service/)。

For services in the same stack, any service is DNS resolvable by it's native `service_name`, if you so wish, you can use links present this service under another alias.

对于同一`stack`，任何服务都是由本机可解析的DNS的`service_name`，如果您愿意，可以使用别名提供此服务的链接。

##### 例子  `docker-compose.yml`

```yaml
version: '2'
services:
  web:
    labels:
      io.rancher.container.pull_image: always
    tty: true
    image: sdelements/lets-chat
    links:
    - database:mongo
    stdin_open: true
  database:
    labels:
      io.rancher.container.pull_image: always
    tty: true
    image: mongo
    stdin_open: true
```
<br>

在这个例子中，`database`可以解析为`mongo`。如果没有链接，`database`将解析为`database`来往的网络服务.

For services in a different stack, the service is DNS already resolvable by `service_name.stack_name`. If you'd prefer to use a specific alias for DNS resolution, you can use `external_links` in the `docker-compose.yml`.

对于不同`stack`的服务，服务是DNS已经解析过的`service_name.stack_name`。如果您希望使用特定别名进行DNS解析，则可以在`docker-compose.yml`中使用`external_links`。

##### 例子  `docker-compose.yml`

```yaml
version: '2'
services:
  web:
    image: sdelements/lets-chat
    external_links:
    - alldbs/db1:mongo
```
<br>

在此示例中，`alldbs` stack 中的`db1`服务将连接到`web`服务。在web服务中，`db1`将可解析为`mongo`。没有外部链接时，`db1`将可解析为`db1.alldbs`。

> **注意:** 交叉stack的发现受（被设计过的）环境的限制。不支stack的持跨环境发现。

### 使用 Rancher Compose 添加服务

阅读更多关于[创建 Rancher Compose]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/rancher-compose/)的详情.

We'll set up the same example that we used above in the UI example. To get started, you will need to create a `docker-compose.yml` file and a `rancher-compose.yml` file. With Rancher Compose, we can launch all the services in the application at once. If there is no `rancher-compose.yml` file, then all services will start with a scale of 1 container.
我们将在UI示例中设置与上面我们使用的相同的示例。开始，您将需要创建一个`docker-compose.yml`文件和一个`rancher-compose.yml`文件。使用Rancher Compose，我们可以一次启动应用程序中的所有服务。如果没有`rancher-compose.yml`文件，则所有服务将以1个容器的scale开始。

#### 例子 `docker-compose.yml`

```yaml
version: '2'
services:
  web:
    labels:
      io.rancher.container.pull_image: always
    tty: true
    image: sdelements/lets-chat
    links:
    - database:mongo
    stdin_open: true
  database:
    labels:
      io.rancher.container.pull_image: always
    tty: true
    image: mongo
    stdin_open: true
```

#### 例子 `rancher-compose.yml`

```yaml
# 你想要拓展的效果服务
version: '2'
services:
  web:
    scale: 2
  database:
    scale: 1
```
<br>

创建文件后，可以将服务启动到Rancher服务器。
```bash
#创建并启动一个没有环境变量的服务并选择一个stack
#如果没有提供stack，stack的名称将是命令运行的文件夹名称
#如果该stack没有存在于Rancher中，它将会被创建
$ rancher-compose --url URL_of_Rancher --access-key <username_of_environment_api_key> --secret-key <password_of_environment_api_key> -p LetsChatApp up -d

#创建并运行一个已经设置好环境变量的服务
$ rancher-compose -p LetsChatApp up -d
```

### Sidekick 服务

Rancher通过允许用户通过使用sidekicks的概念来对这些服务进行分组，从而支持一组服务的托管，调度和锁步。通常创建具有一个或多个配对的服务，以支持容器之间的共享卷（即`--volumes_from`）和网络（即`--net=container`）。



有了服务，你可能希望你的服务的使用`volumes_from`和`net`去连接其他服务。为了使这些起作用，你需要建立一个sidekick关系。通过sidekick关系，Rancher将这些服务作为一个单位进行拓展和调度。例如：B是A的sidekick，服务将始终将A和B作为一对部署，服务的拓展将始终保持一致。

如果您有多个服务总是需要部署在同一主机上，也可能想要定义sidekick关系。

当给一个服务定义一个sidekick时，您不需要链接服务，因为sidekick会自动DNS解析到彼此。

当在服务中使用[负载均衡]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-load-balancers/)，而该服务又拥有sidekick的时候，你需要使用主服务作为负载均衡器的目标。sidekick**不能**成为目标。
Read more about [Rancher's internal DNS]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/internal-dns-service/).

#### 在UI中添加Sidekicks

要设置一个sidekick关系，你可以点击**+ Add Sidekick Container**按钮，它位于`scale`部分。第一个服务被认为是主服务，后面每个附加的sidekick服务都是辅助服务。

####通过Rancher Compose添加Sidekicks

要设置`sidekick`关系，请向其中一个服务添加标签。标签的关键将是`io.rancher.sidekicks`，该值将是服务端。如果你有多个服务添加为sidekicks，它们应该用逗号分隔。例：`io.rancher.sidekicks: sidekick1, sidekick2, sidekick3`

##### 主服务

无论哪个服务包含sidekick标签都被认为是主要服务，而各个sidekicks被视为辅助服务。主服务的scale将用作sidekick标签中所有服务的scale。如果您的所有服务中的scale不同，则主服务的scale将用于所有服务。

当使用带有服务的负载平衡器时，您只能定位主服务，sidekick**不能**成为目标。


##### Rancher Compose里面的Sidekicks例子:

例子`docker-compose.yml`

```yaml
version: '2'
services:
  test:
    tty: true
    image: ubuntu:14.04.2
    stdin_open: true
    volumes_from:
    - test-data
    labels:
      io.rancher.sidekicks: test-data
  test-data:
    tty: true
    command:
    - cat
    image: ubuntu:14.04.2
    stdin_open: true
```

<br>
例子 `rancher-compose.yml`

```yaml
version: '2'
services:
  test:
    scale: 2
  test-data:
    scale: 2
```

##### Rancher Compose里面的Sidekicks例子:多服务使用来自 `volumes_from`的相同服务

如果您有多个服务，他们将使用相同的容器去做一个`volumes_from（数据卷）`，你可以将第二个服务作为主服务的sidekick，并使用相同的数据容器。由于只有主服务可以作为负载平衡的目标，请确保选择正确的服务作为主服务（即，具有sidekick标签的服务）。
Example `docker-compose.yml`

```yaml
version: '2'
services:
  test-data:
    tty: true
    command:
    - cat
    image: ubuntu:14.04.2
    stdin_open: true
  test1:
    tty: true
    image: ubuntu:14.04.2
    stdin_open: true
    labels:
      io.rancher.sidekicks: test-data, test2
    volumes_from:
    - test-data
  test2:
    tty: true
    image: ubuntu:14.04.2
    stdin_open: true
    volumes_from:
    - test-data
```
