---
title: FAQS about Rancher Server
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
redirect_from:
  - /rancher/latest/en/faqs/server/
---

## FAQs 关于 Rancher Server
---

### 我正在运行的 Rancher 是什么版本的?

Rancher的版本位于页面的页脚。 如果你点击该版本，你将可以获得其他组件详细的版本。

### 我怎么样在代理服务器后运行一个 Rancher Server?


请参照 [install Rancher server behind a proxy]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/installing-rancher/installing-server/#launching-rancher-server-behind-a-http-proxy).

<a id="server-logs"></a>

### 我在那能找到 Rancher Server 容器的详细日志？

在Rancher Server上运行`docker logs`将提供一组基本日志。 要获取更详细的日志，你可以进入到Rancher Server容器内部并查看日志文件。

```bash
# 进入 Rancher　Server　容器内部
$ docker exec -it <container_id> bash

# 跳转到 Cattle 日志所在的目录下
$ cd /var/lib/cattle/logs/
$ cat cattle-debug.log
```

在这个目录里面会出现`cattle-debug.log`和`cattle-error.log`。 如果你长时间使用此 Rancher Server，你会发现我们每天都会创建一个新的日志文件。
Inside this folder, there will be `cattle-debug.log` and `cattle-error.log`. If you have been using Rancher server for many days, you will find a log file for each day as we create a new file for each day.

#### 将 Rancher Server 的日志复制到主机上。

以下是将Rancher Server日志从容器复制到主机的命令。

```bash
$ docker cp <container_id>:/var/lib/cattle/logs /local/path
```

### 如何到处 Rancher Server容器的内部数据库？

你可以使用简单的Docker命令从Rancher服务器容器导出数据库。

```
$ docker exec <CONTAINER_ID_OF_SERVER> mysqldump cattle > dump.sql
```

### 如果Rancher Server的IP更改了会怎么样？

如果更改了 Rancher Server 的 IP 地址，你需要重新调整主机更新相关信息。

在 Rancher 中，点击 **系统管理**　> **系统设置**　更新 Rancher Server　URL　**主机注册地址**。注意必须包括 Rancher Server 暴露的端口号。默认情况下我们建议按照安装手册中使用 8080 端口。


主机注册更新后，进入**基础架构** - >**添加主机**- >**自定义**。 `Docker run`命令添加Rancher代理将会更新新的信息。 使用更新的命令，在所有主机上运行命令加入到 Rancher Server中。

###为什么Go-Machine-Service在我的日志中不断重新启动？ 我该怎么办？

Go-machine-service是一种通过websocket连接连接到Rancher API服务器的微服务。 如果无法连接，则会重新启动并再次尝试。

如果你运行在单节点中，它将使用你为主机注册设置的URL连接到Rancher API服务。 验证从Rancher-sever容器内部到达主机注册URL。

```bash
$ docker exec -it <rancher-server_container_id> bash
# 在 Rancher-Server 容器内
$ curl -i <Host Registration URL you set in UI>/v1
```
你应该得到一个json响应。 如果认证开启，响应代码应为401.如果认证未打开，则响应代码应为200。

验证Rancher API Server 能够使用这些变量，通过登录go-machine-service容器并使用你提供给容器的参数进行`curl`命令来验证连接:

```bash
$ docker exec -it <go-machine-service_container_id> bash
# 在go-machine-service 容器内
$ curl -i -u '<value of CATTLE_ACCESS_KEY>:<value of CATTLE_SECRET_KEY>' <value of CATTLE_URL>
```

你应该得到一个json响应和200个响应代码。
You should get a json response back and a 200 response code.

如果 curl 命令失败，那么在`go-machine-service`和Rancher API server之间存在链接问题。


如果curl命令没有失败，则问题可能与go-machine-service尝试建立websocket连接而不是普通的http连接。 如果在go-machine-service和Rancher API服务器之间有一个代理或负载平衡器，请验证它是否配置为允许websocket连接。


### Rancher Server 在运行中变的极慢，怎么去恢复它？

很可能有一些任务由于某些原因而处于僵死状态，如果你能够用界面查看**系统管理** -> **系统进程**，你将可以看到`Running`中的内容，如果这些任务长时间运行（并且失败），则Rancher会最终使用太多的内存来跟踪任务。 本质上为Rancher Server 创建了一个内存不足的状态。

为了使服务器处于响应状态，你需要添加更多内存。 通常，大于4GB的内存就够了。

如果你再次运行 Rancher Server 命令只需要添加一个额外的选项`-e JAVA_OPTS="-Xmx4096m"`

```bash
$ docker run -d -p 8080:8080 --restart=unless-stopped -e JAVA_OPTS="-Xmx4096m" rancher/server
```

根据MySQL数据库的设置方式，你可能需要进行升级才能添加附加命令。

如果由于缺少内存而无法看到**系统管理** - > **系统进程**选项卡，则在有足够内存的前提下重启 Rancher Server就可以看到选项卡并开始对运行时间最长的进程进行故障排错。


###  Rancher Server数据库数据增长太快.

Rancher服务器自动清理几个数据库表，以防止数据库增长太快。如果你注意到这些表没有被及时清理，请随时使用我们的API更新默认设置。

在默认情况下，`container_event` 和 `service_event` 在2 周前产生的任何数据将会被删除。在 API 中的设置以秒为单位被列出(`1209600`)。API 中的设置为`events.purge.after.seconds`.


默认情况下，`process_instance`表在 1 天前产生的数据将会被删除，在 API 中的设置以秒为单位被列出(`86400`)。API 中的设置为`process_instance.purge.after.seconds`.

为了更新你 API 中的设置，跳转到`http://<rancher-server-ip>:8080/v1/settings`页面， 搜索要更新的设置，点击`links -> self` 跳转到你点击的链接去设置，点击侧面的“编辑”更改'值'。 请记住，值是以秒为单位。


<a id="databaselock"></a>

### 为什么 Rancher Server 升级失败或被冻结？

如果你刚开始运行 Rancher 并发现它被永久冻结，也许是liquibase 数据库上锁了。在启动时，liquibase执行模式迁移。它的竞争条件可能会留下一个锁定条目，这将阻止后续的流程。

如果你刚刚升级，在Rancher　Server日志中，MySQL数据库可能存在尚未释放的日志锁定。

```bash
....liquibase.exception.LockException: Could not acquire change log lock. Currently locked by <container_ID>
```

#### 释放数据库锁

> **注意:** 请不要释放数据库锁，除非有相关日志锁的 **异常** 。如果由于数据迁移而升级需要很长时间，那么如果尝试释放数据库锁，你可能会遇到其他迁移问题。

如果你已根据升级文档创建了Rancher Server的数据容器，你需要`exec`到`rancher-data`容器中升级`DATABASECHANGELOGLOCK`表并移除锁，如果你没有创建数据容器，你用`exec`到包含有你数据库的容器中。

```bash
$ sudo docker exec -it <container_id> mysql
```

一旦进入到 Mysql 数据库, 你就要访问`cattle`数据库。

```bash
mysql> use cattle;

#检查表中是否有锁
mysql> select * from DATABASECHANGELOGLOCK;

# 更新移除容器的锁
mysql> update DATABASECHANGELOGLOCK set LOCKED="", LOCKGRANTED=null, LOCKEDBY=null where ID=1;


# 检查锁已被删除
mysql> select * from DATABASECHANGELOGLOCK;
+----+--------+-------------+----------+
| ID | LOCKED | LOCKGRANTED | LOCKEDBY |
+----+--------+-------------+----------+
|  1 |        | NULL        | NULL     |
+----+--------+-------------+----------+
1 row in set (0.00 sec)
```
