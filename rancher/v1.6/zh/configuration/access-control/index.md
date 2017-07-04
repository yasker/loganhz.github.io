---
title: Access Control in Rancher
layout: rancher-default-v1.6
version: v1.6
lang: zh
redirect_from:
  - /rancher/latest/zh/configuration/access-control/
---

## 访问控制
---

### 什么是访问控制？

访问控制是Rancher如何限制用户对Rancher实例的访问权限。默认情况下，Rancher没有配置访问控制，这意味着任何拥有Rancher实例IP地址的人都可以使用它并访问该API，这样使得您的Rancher实例对外开放！我们强烈建议您在启动Rancher后立即配置访问控制，这样您可以按你所想分享你的Rancher实例。他们在访问您的rancher实例之前，需要进行身份验证，只用使用有效的API密钥才能访问您的Rancher API。

Rancher认证的第一个账户将成为 **admin** 账户。 想要获取有关详细信息，请参阅 [admin的权限]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control/#admin).

### 启用访问控制

在 **系统管理** 选项卡中, 点击**访问控制**。

在您的Rancher实例认证后，访问控制将被视为启用。启用访问控制后，您将能够管理不同的 [环境]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/) 并与之分享给其他人。

当访问控制启用后，API被锁定，这时需要用户进行身份验证， 或者使用 [API 密钥]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/api/api-keys/) 来访问它。

#### 活动目录

选择**活动目录**图标。 如果您想要通过TLS来使用活动目录，请确保您具有[使用相应证书来启动Rancher服务器]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/installing-rancher/installing-server/#ldap). 通过点击**身份认证**填写对应信息并验证Rancher。 当活动目录启用后，您将自动以已验证的用户名身份登录。 您将能够作为管理员登录Rancher。

##### 用户搜索与用户组搜索

在配置活动目录时，您将需要输入用户的搜索分支。 此搜索分支允许Rancher搜索在活动目录中已设置的用户。如果您的用户和用户组位于搜索分支中，那么您**仅仅**需要填写用户的搜索分支，但是如果您的用户组在不同的搜索分支，你可以把用户搜索分支替换在`用户组搜索`字段下。 此字段专用于用户组搜索，但不是必需的。

#### Azure AD

选择**Azure AD**图标。 通过单击**Azure验证**，填写相应信息并验证Rancher。 当活动目录启用后，您将自动以已验证的用户名身份登录，您将能够作为管理员登录Rancher。

#### GitHub

选择**GitHub**图标，并按照用户界面中的说明将Rancher注册为GitHub应用程序。 点击**使用GitHub进行身份验证**后，访问控制功能被启用，您将自动使用GitHub登录凭据登录到Rancher并且成为Rancher管理员。

#### 本地身份认证

本地身份验证允许您创建自己的一组账户，这些账户存储在Rancher数据库中。

选择**本地**图标。 通过提供**登录用户名**，**全名**和**密码**来创建管理员用户。 点击**启用本地验证**来启用本地身份验证。 通过单击此按钮，管理员用户将被创建并保存在数据库中。这时您将自动用刚刚创建的管理员帐户登录到Rancher实例。

#### OpenLDAP

 填写对应信息后，通过点击**身份认证**来进行Rancher认证。当OpenLDAP启用后，您将自动以已验证的用户名身份登录，并成为Rancher管理员。

##### 用户搜索与用户组搜索

在配置活动目录时，您将需要输入用户的搜索分支。 此搜索分支允许Rancher搜索在活动目录中已设置的用户。如果您的用户和用户组位于搜索分支中，那么您**仅仅**需要填写用户的搜索分支，但是如果您的用户组在不同的搜索分支，你可以把用户搜索分支替换在`用户组搜索`字段下。 此字段专用于用户组搜索，但不是必需的。

#### Shibboleth

选择**Shibboleth**图标。 填写Shibboleth帐户的配置信息，点击**保存**保存信息，然后点击**测试**来测试访问控制是否正常工作。

使用Shibboleth时，如果您正在配置来验证它，您应该注意一些已知的问题。

* 没有搜索或查找支持功能。 在添加用户时，想要正确的用户获取访问权限必须输入确切的ID。
* 当添加用户到一个[环境]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/)时, 不支持组ID，除非管理员打开该组成员的访问控制权限。

### 站点访问

根据您的身份验证类型，Rancher提供不同级别的站点访问。

#### 活动目录/GitHub/Shibboleth

如果您已通过AD或GitHub进行身份验证，则将有3个选项可用。

* **允许任何有效的用户** - GitHub或活动目录中的任何用户都可以访问您的Rancher实例。 **不**推荐将此设置用于GitHub，因为它将使GitHub中的任何用户都可以访问Rancher实例。
* **允许环境成员，加上已授权用户和组织** - 一个环境的成员用户或拥有者用户和添加到`已授权用户和用户组`的用户一样，都有权限访问Rancher实例。
* **限制访问只有授权用户和用户组可以访问** - 只有添加到`已授权用户和用户组`的用户才能访问Rancher实例。 即使用户已被添加到环境中，如果没有被添加到`已授权用户和用户组`，他们将仍然无法访问Rancher实例。

任何具有Rancher实例权限的人都将被授予 [用户]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/accounts/#users)权限. 他们将无法查看**管理员**标签。 如果想要他们查看，您将需要明确地将其帐户更改为[管理员帐户]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/accounts/#admin)。

为了让用户查看不同的[环境]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/), 它们将需要被环境的[所有者][owner]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/#owners)添加到环境中。


#### Azure AD/OpenLDAP

对于Azure AD和OpenLDAP，您的设置中的任何用户都可以访问Rancher站点。

#### 本地身份认证

启用本地身份认证后，管理员可以通过访问**管理员**> **帐户**选项卡来创建其他管理员/用户。 点击**添加帐户**并填写您要添加的帐户的详细信息。 您可以选择其帐户类型为**管理员**或**用户**。 管理员可以查看“**管理员**标签，而Rancher实例的其他用户将无法看到该选项卡。

一旦创建一个帐户后，该账户可以被添加到任何[环境]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/)中。

### 账户类型

帐户类型决定帐户是否可以访问管理员选项卡。对于Rancher中的每个环境，可以提供不同级别的[成员角色]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/#membership-roles)来对特定环境进行访问。

#### 管理员

认证Rancher的第一个用户成为Rancher的管理员。 只有管理员才有权限查看**管理员**标签。

在管理环境时，管理员可以查看Rancher中的所有[环境]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/environments/)， 即使管理员没有被加入到该环境的成员中。 在管理员环境下拉菜单中，成员只能看到他们在成员列表中的环境。

管理员可以将其他用户添加为Rancher管理员。 在用户登录Rancher后，他们可以在 **系统管理** > **账户**页面上更改用户角色。 在**管理员**> **帐户**标签中，点击帐户名称旁边的**编辑**，并将帐户类型更改为管理员。 点击**保存**。

#### 用户

除了Rancher的认证用户外，任何其他用户都将自动添加用户权限。 他们将无法看到**管理员**标签。

他们只能看到他们同组成员的环境。

### 禁用访问控制

如果您决定不再需要访问控制，请单击**禁用访问控制**按钮。 这将使您的Rancher实例向公众开放，任何人都可以访问您的API。 这是**不**推荐。

### 配置会话超时

默认情况下，会话在其创建后16小时到期。 如果您觉得时间太长，可以更新会话到期时间。

1. 点击**系统管理** -> **系统设置** -> **Advanced Settings**, 点击 **I understand that I can break things by changing advanced settings**。
2. 找到 **api.auth.jwt.token.expiry**设置，然后点击edit按钮。
3. 更新超时会话值后，然后单击**保存**按钮。 值以毫秒为单位。

