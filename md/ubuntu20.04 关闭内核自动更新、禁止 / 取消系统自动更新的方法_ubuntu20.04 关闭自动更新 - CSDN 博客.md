> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/inthat/article/details/125316389)

#### 文章目录

*   *   [ubuntu20.04 关闭内核自动更新](#ubuntu2004_1)
    *   [ubuntu 禁止 / 取消系统自动更新的方法](#ubuntu__45)

### ubuntu20.04 关闭内核自动更新

问题背景：  
ubuntu 默认启动了自动更新内核，我们可以进一步关闭内核更新，使用当前内核。

解决方法：

```
cat /etc/issue
uname -r

```

查看安装内核

```
dpkg --list | grep linux-image
dpkg --list | grep linux-headers
dpkg --list | grep linux-modules

```

【推荐】方案一：  
安全禁止 [ubuntu 更新](https://so.csdn.net/so/search?q=ubuntu%E6%9B%B4%E6%96%B0&spm=1001.2101.3001.7020)

```
vi /etc/apt/apt.conf.d/10periodic
vi /etc/apt/apt.conf.d/20auto-upgrades

```

后面部分全部改成 “0”

```
reboot

```

方案二：hold – 用户希望软件包保持现状，例如，用户希望保持当前的版本，当前的状态，当前的一切。  
直接使用 hold 参数，固定内核版本：

```
# apt-mark hold linux-image-5.4.0-70-generic
linux-image-5.4.0-70-generic set on hold.
# apt-mark hold linux-headers-5.4.0-70-generic
linux-headers-5.4.0-70-generic set on hold.
# apt-mark hold linux-modules-extra-5.4.0-70-generic
linux-modules-extra-5.4.0-70-generic set on hold.

```

查询 [Ubuntu 系统](https://so.csdn.net/so/search?q=Ubuntu%E7%B3%BB%E7%BB%9F&spm=1001.2101.3001.7020)被锁定不更新的软件包状态 (hold), 命令为:

```
sudo dpkg --get-selections | grep hold 

```

### ubuntu 禁止 / 取消系统自动更新的方法

1. 打开 “Software & Updates（软件和更新）”，如下所示。  
![](https://i-blog.csdnimg.cn/blog_migrate/a6ec78432531dccf21f22354ba34ab89.png)在界面上的 “Update（更新）” 选项卡中，如果要 开启 自动更新按如下界面进行设置。  
![](https://i-blog.csdnimg.cn/blog_migrate/9e9cb2dc348f9b0672ce48b15e0a78f6.png)