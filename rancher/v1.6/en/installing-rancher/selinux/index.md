---
title: Using Rancher with SELinux Enabled
layout: rancher-default-v1.6
version: v1.6
lang: zh
---

## 在SELinux模式下使用Rancher - RHEL/CentOS
---

_Available for Rancher 1.6+_

如果你的Rancher是运行在RHEL/CentOS并想启用SELinux，你需要安装额外的SELinux模块

这个文档的步骤是一个必须的临时措施，但在RHEL以及CentOS安装升级[changes present in the module](https://github.com/projectatomic/container-selinux/pull/33)后已经不需要了。一旦这个更新被应用，以下的步骤则不再需要了。

这些步骤必须在Rancher server容器所在的主机以及所有的[hosts]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/hosts/)上运行。

### 安装包并构建SELinux模块

为了安装另外的SELinux模块，你需要安装 `selinux-policy-devel` 这个包。

```bash
$ yum install selinux-policy-devel
```

### 构建模块

创建一个名叫 `virtpatch.te` 的文件并写入以下内容

```
policy_module(virtpatch, 1.0)

gen_require(`
  type svirt_lxc_net_t;
')

allow svirt_lxc_net_t self:netlink_xfrm_socket create_netlink_socket_perms;
```

构建模块

```
$ make -f /usr/share/selinux/devel/Makefile
```

After running the `make` command, a file named `virtpatch.pp` should be created if the build was successful. `virtpatch.pp` is the compiled SELinux module.
运行 `make` 命令后，当构建成功后，一个名叫 `virtpatch.pp` 的文件将会被创建。`virtpatch.pp` 是编译后的SELinux模块

### 加载模块

在SELinux模块被构建后，使用以下命令加载。

```
# Load the module
$ semodule -i virtpatch.pp
# Verify the module is loaded
$ semodule -l
```
