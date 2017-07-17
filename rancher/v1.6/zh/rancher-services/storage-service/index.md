---
title: Persistent Storage Service in Rancher
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---

## 存储服务
---

Rancher提供了不同的存储服务，使用户可以将卷存储映射给容器。

### 配置存储服务

当我们创建[环境模板]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/#what-is-an-environment-template)时，用户可以从应用商店选择需要在环境中的使用存储服务。

或者，如果用户已经创建了一个环境，你可以从 [应用商店]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/catalog/)中选择并启动一个存储服务。

> **注意:** 某些存储服务可能无法和一些容器或编排调度系统（例如，kubernetes）所兼容。环境模板可以根据当前的编排调度系统限定可以使用的存储服务，而应用商店中则会显示全部的存储服务。

### 查看存储驱动

在存储服务启动后，在**基础架构** -> **存储**的界面中可以看到一个存储驱动已经被创建出来。在这个界面中，用户可以查看当前环境中所有可用的存储驱动。存储驱动的名称和刚刚启动的存储服务stack的名称保持一致。

对于每一种存储驱动，主机上运行的存储服务都会被显示出来。正常情况下，环境里的所有主机都会出现在该页面。同时，存储驱动提供的卷列表以及卷的状态也会被显示出来。你可以看到每个卷的名称（比如，主机上的卷名称），以及每个卷的挂载点。对于每个挂载点，其容器信息以及该挂载点在容器中映射的路径都会被显示出来。

### 卷的作用域

Rancher的存储服务中，卷的作用域可以在不同的级别生效。目前，只有[Rancher Compose](#using-storage-drivers-with-rancher-compose)支持创建不同的存储作用域。UI上仅仅支持环境级别的卷创建操作。

#### 应用级别

应用级别的存储卷，应用中的服务如果引用了相同的卷都将共享同一个存储卷。不在该应用内的服务则无法使用此存储卷。

Rancher中，应用级别的存储卷的命名规则为使用应用名称为前缀来表明该卷为此应用独有，且会以随机生成的一组数字结尾以确保没有重名。在引用该存储卷时，你依然可以使用原来的卷名称。比如，如果你在应用 `stackA`中创建了一个名为`foo` 的卷， 你的主机上显示的卷名称将为`stackA_foo_<randomNumber>`，但在你的服务中，你依然可以使用`foo`。

<!-- #### Container Scoped

With a container scoped volume, a new volume is created for each instance of a container. No containers would share the same volume.

In Rancher, container scoped volumes are prefixed with `containerscoped` and have a suffix of the `<dockerContainerID>` and a random number to guarantee no duplication. For example, if you create a volume called `foo1` with a container scope, the volume name in the UI and on your hosts will be `containerscoped_foo1_1_<dockerContainerID>_<randomNumber>`. -->

#### 环境级别

环境级别的存储卷，该环境中的所有服务如果 引用了相同的卷将共享同一个存储卷。不同应用中的不同服务也可以共享同一个存储卷。目前，环境级别的卷只可以通过UI创建。

### 在UI中使用存储驱动

在你的存储服务启动后且状态为`active`，使用共享存储卷的[services]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-services/)就可以被创建了。在创建服务时，在**卷**选项卡中，输入**卷**以及**卷驱动**

**卷** 语法和Docker语法相同，`<volume_name>:</path/in/container>`。Docker卷默认挂载为读写模式，但是你可以通过在卷的末尾添加`:ro`将其挂载为只读模式。

**卷驱动**和存储驱动的名字一致，为存储驱动的应用名。

如果 `<volume_name>`在存储驱动中已经存在，在存储卷作用域范围内，将使用相同的存储卷。

#### 创建新卷

一个卷可以被分为两部分创建：

1. 创建服务时，如果 **卷**选项卡中的卷在存储驱动中还不存在，环境级别的存储卷将被创建。如果卷已经存储，将不会再创建新卷。
> **注:**该设定并不适用于Rancher EBS，使用Rancher EBS时，必须首先定义一个卷。

2. 在**基础架构** -> **存储**界面中，选择**添加卷**。输入卷名称以及驱动信息如果你需要的话。该卷在被一个服务使用之前将一直保持 `inactive` 状态。

### 在Rancher Compose中使用存储驱动

在基础设施应用中的存储服务启动后，你可以开始创建卷了。在下面的例子中，我们将使用**Rancher NFS** 存储服务。
在Docker Compose文件中`volumes`下可以定义卷。在同一个Docker Compose中每个卷可以和多个服务关联。此功能只在Compose v2格式下生效。


```yaml
version: '2'
services:
  foo:
    image: busybox
    stdin_open: true
    volumes:
    - bar:/var/lib/storage
volumes:
  bar:
    driver: rancher-nfs
```

#### 应用级别

默认情况下，所有的卷将为应用级别。在同一个Compose文件中所有引用同一个卷的服务或应用将共享同一个卷。

如果再同一个Compose文件中创建了一个新应用，一个新卷也会被创建。当应用被删除时，卷也会被删除。
In the above example, volume `bar` has stack scope.
在上面的例子上，卷`bar`即为应用级别。
<!--
#### Container Scoped

In some cases it makes sense to have a volume created for each instance of a container. This is indicated via the `per_container` option for a volume.

```yaml
version: '2'
services:
  foo:
    image: busybox
    stdin_open: true
    volumes:
    - bar:/var/lib/storage
volumes:
  bar:
    driver: rancher-nfs
    per_container: true
```

When scaling up the `foo` service, a volume will be created for each new container. When scaling down the `foo` service, the volumes corresponding to the removed containers will be removed.
-->

#### Environment Scoped

To use volumes across stacks, you would need to use an environment scoped volume. In this case, volumes must already be created in Rancher prior to starting services and stacks using the volume. To use an environment scoped volume, you'd add the `external` option to the volume.

```yaml
version: '2'
services:
  foo:
    image: busybox
    stdin_open: true
    volumes:
    - bar:/var/lib/storage
volumes:
  bar:
    driver: rancher-nfs
    external: true
```

If a volume by the name of `bar` is not found at the environment level when launching this stack, then an error will be thrown. Environment scoped volumes can only be removed from the UI.
