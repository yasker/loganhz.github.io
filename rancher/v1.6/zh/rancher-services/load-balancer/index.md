---
title: Load Balancers in Rancher
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---

## 负载均衡器
---

Rancher提供在Rancher内使用不同负载均衡器驱动程序的能力。负载均衡器可以通过向目标服务添加规则来将网络和应用程序流量分配到单个容器。任何目标服务将使其所有底层容器自动注册为Rancher的负载平衡器目标。

默认情况下，Rancher使用HAProxy提供了一个托管负载均衡器，可以手动扩容到多个主机。我们计划添加额外的负载均衡器提供者，所有负载均衡器的选项将是相同的，不管负载均衡器提供者。

对于Cattle引擎的环境，可以参考[UI]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-load-balancers/#load-balancer-options-in-the-UI)和[Rancher Compose]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-load-balancers/#load-balancer-options-in-rancher-compose)了解更多信息，并且在[UI]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-load-balancers/#adding-a-load-balancer-in-the-ui)和[Rancher Compose]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/cattle/adding-load-balancers/#adding-a-load-balancer-with-rancher-compose)中有相关的例子。

对于Kubernetes的环境，详细了解如何根据[云提供商]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/kubernetes/providers/)启动外部负载均衡器服务，或者使用[Rancher负载均衡器在Kubernetes环境中进行入口支持]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/kubernetes/ingress/)。
