---
title: Networking in Rancher
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
redirect_from:
  - /rancher/latest/en/rancher-services/networking/
---

## 网络
---
Rancher实现了一个CNI框架，用户可以在Rancher中选择不同的网络驱动。为了支持CNI框架，每个Rancher环境中都需要部署Network Services，默认情况下，每个环境模版都会启用Network Services。除了Network Services这个基础设施服务之外，你还需要选择相关的CNI driver。在默认的环境模版中，IPSec驱动是默认启用的，它是一种简单且有足够安全性的隧道网络模型。当你一个网络驱动在环境中运行时候，它会自动创建一个默认网络，任何使用manged网络的服务其实就是在使用这个默认网络。

### 与先前版本的区别
当使用1.2版本之前的IPsec网络时，容器使用managed网络将会被分配两个IP，分别是Docker bridge IP（172.17.0.0/16）和Rancher managed IP（10.42.0.0/16）。之后的版本中，则集成了CNI网络框架的标准，容器只会被分配Rancher managed IP（10.42.0.0/16）。

### Implications of using CNI
Rancher managed IP不会显示在Docker元数据中，这意味着通过docker inspect无法查到IP。任何端口映射也无法通过docker ps显示出来，因为Rancher使用IPtables来管理端口映射。

###容器间连通性
默认情况下，同一环境下的managed网络之间的容器是可达的。如果你想要控制这个行为，你可以部署network policy。
如果你在跨主机容器通信中碰到问题，可以移步troubleshooting文档。

### 网络类型
在UI上创建服务时，切换到“Networking”Tab页上可以选择网络类型，但是UI上默认不提供“Container”网络类型，如果要使用“Container”类型，则需要通过Rancher CLI/Rancher Compose/Docker CLI来创建。

#### Managed
默认情况下，通过UI创建容器会使用managed网络，在容器中使用ip addr或者ifconfig可以看到eth0和lo设备，eth0的IP从属于Rancher managed子网中，默认的子网是10.42.0.0/16，当然你也可以修改这个子网。
注意：如果在基础设施服务中删除了网络驱动服务，那么容器的网络设置将会失效。

##### 通过Docker CLI创建容器
任何通过Docker CLI创建的容器，只要添加--label io.rancher.container.network=true的标签，那么将会自动使用managed网络。不用这个标签，大部分情况下使用的是bridge网络。
如果容器只想使用managed网络，你需要使用--net=none和--label io.rancher.container.network=true。

#### None
当容器使用none网络类型，基本上等同于Docker中的—net=none。在容器中也不会看到任何网络设备除了lo设备。

#### Host
当容器使用host网络类型，基本上等同于Docker中的—net=host。在容器中能够看到主机的网络设备。

#### Bridge
当容器使用bridge网络类型，基本上等同于Docker中的—net=bridge。默认情况下，容器中可以看到172.17.0.0/16的网段IP。

#### Container
当容器使用container网络类型，基本上等同于Docker中—net=container:<CONTAINER>。在容器中可以看到指定容器的网络配置。

### Rancher IPSEC使用例子
通过编写YAML文件，利用CNI框架来驱动，就可以构建Rancher的网络基础服务。下面是IPSEC网络驱动的YAML文件样例：
```
ipsec:
  network_driver:
    name: Rancher IPsec
    default_network:
      name: ipsec
      host_ports: true
      subnets:
      - network_address: $SUBNET
      dns:
      - 169.254.169.250
      dns_search:
      - rancher.internal
    cni_config:
      '10-rancher.conf':
        name: rancher-cni-network
        type: rancher-bridge
        bridge: $DOCKER_BRIDGE
        bridgeSubnet: $SUBNET
        logToFile: /var/log/rancher-cni.log
        isDebugLevel: ${RANCHER_DEBUG}
        isDefaultGateway: true
        hostNat: true
        hairpinMode: true
        mtu: ${MTU}
        linkMTUOverhead: 98
        ipam:
          type: rancher-cni-ipam
          logToFile: /var/log/rancher-cni.log
          isDebugLevel: ${RANCHER_DEBUG}
          routes:
          - dst: 169.254.169.250/32
```

#### Name
网络驱动的名字

#### Default Network
默认网络定义的是当前Environment的网络配置

##### Host Ports
默认情况下，可以在主机上开放端口，当然你可以选择不开放

##### Subnets
你可以给Overlay网络定义一个子网

##### DNS && DNS Search
这两个配置Rancher会自动生成

#### CNI 配置
你可以将CNI的具体配置放在`cni_config`下面，具体的配置将会依赖你选择的CNI插件

##### bridge
Rancher IPSEC实际上利用了CNI的bridge插件，所以你会看到这个设置，默认是docker0

##### bridgeSubnet
这个配置可以理解为主机上容器的子网，对于Rancher IPSEC就是10.42.0.0/16

##### mtu
不同的网络环境MTU的配置可能会不同，尤其是当你使用Overlay网络更需要注意。最终容器的MTU值应该是网络的MTU减去Overlay网络overhead的大小。比如IPSEC：1500-98=1402
