---
title: Adding Custom Hosts
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---

##添加Custom主机
---

如果你已经部署了Linux主机，并且希望将它们添加到Rancher，点击**Custom**图标。之后会为你自动生成一份`docker`命令脚本，将其拷贝到每一台主机上运行以启动`rancher/agent`容器。

如果你的[环境]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/)不一样，那么生成的命令也会是独一无二的，无论你所在的是哪个环境。

确保你所在的环境就是你想要添加主机的环境。你所在的环境显示在左上角。当你第一次登录进去之后，你是处于**默认**环境的。

一旦你的主机添加到Rancher之后，它们就可以[添加服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-services/)。

> **注意：**确保运行Rancher服务器的主机和你所添加的主机的时间戳是一样并且能够正常访问主机上的容器。更多信息，请参考[Rancher访问一个容器的shell和logs]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/faqs/troubleshooting/#container-access)。

###主机标签

每台主机，你都可以添加标签，以帮助你组织你的主机。所添加的标签会作为rancher／agent容器启动时候的环境变量。添加的标签是一组key/value对，并且密钥必须是唯一识别符。如果你添加了两个具有不同值的密钥，那么将会以你后面添加的值为准作为key/value对。

增加标签之后，你可以使用这些标签[调度服务/负载均衡器]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/scheduling/)，并且可以为你主机上的[服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-services/)创建黑白名单。

添加custom主机的时候，你可以在UI上添加标签，之后将会自动将带有key/value的环境变量(`CATTLE_HOST_LABELS`)添加到UI上出现的命令里。

_例子_

```bash
# Adding one host label to the rancher/agent command
$  sudo docker run -e CATTLE_HOST_LABELS='foo=bar' -d --privileged \
-v /var/run/docker.sock:/var/run/docker.sock rancher/agent:v0.8.2 \
http://<rancher-server-ip>:8080/v1/projects/1a5/scripts/<registrationToken>

# Adding more than one host label requires joining the additional host labels with an `&`
$  sudo docker run -e CATTLE_HOST_LABELS='foo=bar&hello=world' -d --privileged \
-v /var/run/docker.sock:/var/run/docker.sock rancher/agent:v0.8.2 \
http://<rancher-server-ip>:8080/v1/projects/1a5/scripts/<registrationToken>
```

### 安全组／防火墙

对于添加的任何主机，请确保任何组或者防火墙允许流量经过，否则Rancher的功能将会受限。

* 如果你正在使用IPsec[网络驱动]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/networking/)，与其他所有主机之间UDP端口500和4500。
* 如果你正在使用VXLAN[网络驱动]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/networking/)，与其他所有主机之间UDP端口`4789`。
* _k8s主机：_用作K8s的主机需要开放`10250`和`10255`端口作为`kubectl`使用。为了访问外部的服务，NodePort使用的端口也需要开放，默认的是TCP端口`30000` - `32767`。

<a id="samehost"></a>

### 在运行Rancher Server的机器上添加主机

如果你将主机添加到正在运行Rancher Server的机器上，你必须重新编辑UI上提供的命令。在UI上，你可以指定用于注册这台主机的IP，它将会作为环境变量自动添加到命令行中。

```bash
$ sudo docker run -d -e CATTLE_AGENT_IP=<IP_OF_RANCHER_SERVER> -v /var/run/docker....
```

如果你已经在运行Rancher Server的机器上添加了一台主机，注意由于Rancher Server的UI需要依赖`8080`端口，所以主机上那些绑定端口`8080`的容器将无法创建，否则将会出现端口冲突的情况，造成Rancher Server将停止。如果你一定要使用`8080`端口，那么你需要在启用Rancher Server的时候用另一个端口。

### 通过代理的主机

为了设置一个HTTP代理， 将这个代理添加到docker环境中。在添加custom主机之前，修改文件`/etc/default/docker`，指向你的代理并重新启动docker。

```bash
$ sudo vi /etc/default/docker
```

在文件里，编辑`#export http_proxy="http://127.0.0.1:3128/"`，使其指向你的代理。保存你的修改并且重启docker。在不同的系统上重启docker的方式是不一样的。

> **注意：** 如果是用systemd启动的docker, 那么注册http代理的方式请参考docker [介绍](https://docs.docker.com/articles/systemd/#http-proxy)。
另外没有其他的环境变量加在命令行中去启动Rancher agent。你只需要保证你的docker daemon正常配置。

### 同时包含私有IP和公共IP的虚拟机

默认情况下，对于同时包含私有IP和公共IP的虚拟机，IP地址将会根据主机注册地址中指定的地址来确定。例如，如果注册地址中使用的是私有IP，那么就会使用主机的私有IP。如果你想修改主机的IP地址，你需要编辑UI中提供的命令行。为了使Rancher agent中的容器能够正常启动，需要将环境变量`CATTLE_AGENT_IP`设置成期望的IP地址。Rancher中所有的主机都需要和Rancher Serve在同一个网络。

```bash
$ sudo docker run -d -e CATTLE_AGENT_IP=<PRIVATE_IP> -v /var/run/docker....
```
如果在agent已经连接之后需要修改主机的IP，那么请重新运行custom命令行。

> **注意：** 当设置成私有地址之后，Rancher中任何已经存在的容器将不作为同一个管理网络的一部分。
