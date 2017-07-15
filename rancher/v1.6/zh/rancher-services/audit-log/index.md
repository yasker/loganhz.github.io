---
title: Audit Logging in Rancher
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---

## 审计日志
---

只有[admins]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control/#admin)用户有权限访问审计日志。审计日志在**系统管理**->**审计日志**。

Rancher的审计日志是不同事件类型的集合。

* 任何带有前缀`api`的事件是API的一次调用。事件类型将记录API操作，谁执行的操作以及API调用的方式（即通过UI，通过API密钥）。
* 任何没有带`api`前缀的事件都是Rancher服务器正在做的事情。例如，在协调服务的容器期间，可能会创建一个将记录`instance.create`事件的实例。
