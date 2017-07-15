---
title: Accounts in Rancher
layout: rancher-default-v1.6-zh
version: v1.6
lang: zh
---

## 账户
---

### 什么是账户?

有权访问Rancher的每个用户都有Rancher帐号。对于本地身份验证设置，你可以为用户创建帐户，对于其他身份验证提供者，当用户登录到Rancher时，会为该用户创建一个帐户。

#### AD域/GitHub/OpenLDAP验证

当[AD域]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control/#active-directory)，[Azure AD]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control/#azure-ad)，[GitHub]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control/#github)，或者[OpenLDAP]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control/#openldap)验证开启的时候，**帐户**选项卡显示已登录并针对Rancher进行身份验证的用户列表。为了登录，它们必须被赋予[访问站点]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control/#site-access)的权限或被添加到[环境]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/)中。

#### 本地验证

[启用本地身份验证]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control/#local-authentication)后，帐户可以在**帐户**选项卡中添加到Rancher。点击**添加帐户**按钮将帐户添加到Rancher数据库。创建帐户时，帐户类型可以指定为管理员或用户。

### 账户类型

帐户类型确定帐户是否可以访问管理员选项卡。对于Rancher中的每个环境，都有一个[成员角色]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/#membership-roles)，为特定环境提供不同的访问级别。

#### 管理员

认证Rancher的第一个用户成为Rancher的管理员。只有管理员才有权查看**管理员**标签。

管理环境时，管理员可以查看Rancher中的所有[环境]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/)，即使管理员未添加为环境成员。在管理员环境下拉菜单中，成员只会看到他们在成员资格列表中的环境。

管理员可以将其他用户添加为Rancher管理员。在用户登录Rancher后，他们可以在**系统管理** > **账号设置**页面更改用户的角色。在**系统管理** > **账号设置**页面中，点击账号旁边的**编辑**，并将账号类型更改为_管理员_。点击**保存**。

#### 用户

除了认证Rancher的用户，任何其他用户都将自动添加用户权限。他们将无法看到**管理员**标签。

他们只能查看他们所属的环境。
