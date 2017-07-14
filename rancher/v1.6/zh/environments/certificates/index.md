---
title: Certificates in Rancher
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---

## 证书
---

### 添加证书

要添加一个证书到你的 [环境]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/)，你需要到 **基础架构** -> **证书** 页面。这个页面列出了所有已经添加到 Rancher 环境的证书。你可以点击 **添加证书** 添加一个新证书。

1. 填写证书 **名称** 和 **描述**。

2. 填写证书 **私钥**。 你可以点击 **从文件读取** 导入文件，或粘体私钥到文本框。

3. 填写 **证书**。 你可以点击 **从文件读取** 导入文件，或粘体私钥到文本框。
4. (可选) 如果你有其它的证书链， 你也可以通过 **从文件读取** 导入文件，或粘体私钥到文本框。

### 使用证书

加载到环境(Environment)的证书可用作[负载均衡的 SSL 终端]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-load-balancers/#ssl-termination) 或 [Kubernetes 入口的 TSL 终端]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/kubernetes/ingress/#example-using-tls).
