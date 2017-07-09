---
title: Service Accounts in Rancher
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---

## Service Accounts
---

如果你创建了一个账号需要和Rancher API交互，需要创建service account API keys，这样我们就可以访问带有权限认证的API。为了在service中创建这些keys，需要添加以下的labels到service

Key | Value |Description
----|-----|---
`io.rancher.container.create_agent` | `true` | 标识service account API keys会被添加到每个container的环境变量里。
`io.rancher.container.agent.role` | `environment` | 标识account的角色。创建service accounts的值为`environment`.


当service中的containers启动时，以下环境变量会被加入到container中


Key| Value
---|---
`CATTLE_URL` | [host registration]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/settings/#host-registration)的URL。
`CATTLE_ACCESS_KEY` | 启动的service所在[environment]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/environments/)access key。
`CATTLE_SECRET_KEY` | Access key对应的secret key。
