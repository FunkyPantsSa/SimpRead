> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bbs.huaweicloud.com](https://bbs.huaweicloud.com/blogs/400849)

> Ubuntu 20.04 内核自动升级会带来系统软件兼容性问题。建议在使用时，禁止自动内核升级，进行手动升级。

### 1. 背景描述

在 Ubuntu 20.04 每次内核升级后，系统需要重新启动以加载新内核。如果您已经安装了自动更新功能，则系统将自动下载和安装可用的更新，这可能导致系统在不经意间被重启，若使用的软件依赖于特定版本的内核，那么当系统自动更新到新的内核版本时，可能会出现兼容性问题。在使用 Ubuntu20.04 时，建议手动控制内核的更新。

### 2. 详细步骤

在 Ubuntu 20.04 上禁止内核自动升级，你可以采取以下步骤：

**1. 禁用 unattended-upgrades：**

`unattended-upgrades` 是一个用于安装安全更新的软件包。要禁用它，首先打开 `/etc/apt/apt.conf.d/20auto-upgrades` 文件：

```
vi /etc/apt/apt.conf.d/20auto-upgrades

```

将其中的 `Unattended-Upgrade "1";` 改为 `Unattended-Upgrade "0";` 以禁用自动更新。保存文件并退出。

**2. 将当前内核版本固定（锁定）：**

要禁止特定的内核版本更新，你可以使用 `apt-mark` 命令将其固定。首先，检查你当前的内核版本：

```
uname -r

```

例如，如果你的内核版本是 `5.4.0-42-generic`，你需要锁定所有与此版本相关的软件包。为此，执行以下命令：

```
sudo apt-mark hold linux-image-5.4.0-42-generic linux-headers-5.4.0-42-generic linux-modules-5.4.0-42-generic linux-modules-extra-5.4.0-42-generic

```

**3. 禁用自动更新：**

要禁用所有自动更新，首先打开 `/etc/apt/apt.conf.d/10periodic` 文件：

```
vi /etc/apt/apt.conf.d/10periodic

```

修改文件以将所有选项设置为 `0`：

```
APT::Periodic::Update-Package-Lists "0";
APT::Periodic::Download-Upgradeable-Packages "0";
APT::Periodic::AutocleanInterval "0";
APT::Periodic::Unattended-Upgrade "0";

```

保存文件并退出。

执行完这些步骤后，你的 Ubuntu 20.04 系统将不会自动升级内核。请注意，禁用自动更新可能会导致你的系统变得不安全，因为你需要手动安装重要的安全补丁。在禁用自动更新之前，请确保你明白其中的风险。

【版权声明】本文为华为云社区用户原创内容，转载时必须标注文章的来源（华为云社区）、文章链接、文章作者等基本信息， 否则作者和本社区有权追究责任。如果您发现本社区中有涉嫌抄袭的内容，欢迎发送邮件进行举报，并提供相关证据，一经查实，本社区将立刻删除涉嫌侵权内容，举报邮箱： [cloudbbs@huaweicloud.com](mailto:cloudbbs@huaweicloud.com)