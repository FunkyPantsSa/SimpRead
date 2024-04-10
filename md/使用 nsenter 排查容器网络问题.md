> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.bilibili.com](https://www.bilibili.com/read/cv33286830/)

需求
==

我想进入容器中执行 curl 命令探测某个地址的连通性，但是容器镜像里默认没有 curl 命令。我这里是一个内网环境不太方便使用 yum 或者 apt 安装，怎么办？

这个需求比较典型，这里教大家一个简单的方法，使用 nsenter 进入容器的 net namespace，即可使用宿主机的 curl、ip、ifconfig 等命令，其效果，就跟进入容器中执行是一样的。

原理
==

容器像是一个轻量级虚拟机，有自己的 IP，宿主机如果已经监听了 8080 端口，容器里的进程仍然可以重复监听 8080 端口。核心就是因为容器有自己的 namespace，和宿主机的 net namespace 互不影响。关于容器的 namespace 相关知识，可以 Google 一下关键字：「容器 namespace 原理」。

nsenter，是个关键工具。其用法如下：

```
nsenter --help

用法：
 nsenter [选项] [<程序> [<参数>...]]

以其他程序的名字空间运行某个程序。

选项：
 -a, --all              enter all namespaces
 -t, --target <pid>     要获取名字空间的目标进程
 -m, --mount[=<文件>]   进入 mount 名字空间
 -u, --uts[=<文件>]     进入 UTS 名字空间(主机名等)
 -i, --ipc[=<文件>]     进入 System V IPC 名字空间
 -n, --net[=<文件>]     进入网络名字空间
 -p, --pid[=<文件>]     进入 pid 名字空间
 -C, --cgroup[=<文件>]  进入 cgroup 名字空间
 -U, --user[=<文件>]    进入用户名字空间
 -S, --setuid <uid>     设置进入空间中的 uid
 -G, --setgid <gid>     设置进入名字空间中的 gid
     --preserve-credentials 不干涉 uid 或 gid
 -r, --root[=<目录>]     设置根目录
 -w, --wd[=<dir>]       设置工作目录
 -F, --no-fork          执行 <程序> 前不 fork
 -Z, --follow-context  根据 --target PID 设置 SELinux 环境

 -h, --help             display this help
 -V, --version          display version

更多信息请参阅 nsenter(1)。

```

通过 `nsenter -t <pid> -n bash` 即可进入 pid 指向的进程的 net namespace，并在这个 namespace 执行 bash 命令，之后，在这个 bash session 里执行 curl、ip、ifconfig 等命令，看到的网络信息就都是容器内部的网络信息。

实战
==

在 Linux 上，通过夜莺的 docker-compose 拉起一套夜莺，使用 bridge 模式，做个演示。

1. 拉起夜莺
=======

```
[root@aliyun-2c2g40g3m compose-bridge]# docker compose up -d
[+] Running 7/7
 ✔ Network compose-bridge_nightingale  Created                                                                                                                                                                                                                                                                                     0.1s
 ✔ Container redis                     Started                                                                                                                                                                                                                                                                                     0.1s
 ✔ Container victoriametrics           Started                                                                                                                                                                                                                                                                                     0.1s
 ✔ Container mysql                     Started                                                                                                                                                                                                                                                                                     0.1s
 ✔ Container ibex                      Started                                                                                                                                                                                                                                                                                     0.1s
 ✔ Container nightingale               Started                                                                                                                                                                                                                                                                                     0.1s
 ✔ Container categraf                  Started

```

然后，inspect 一下 nightingale 容器，拿到其 Pid：

```
[root@aliyun-2c2g40g3m compose-bridge]# docker ps |grep nigh
0500a886538e   flashcatcloud/nightingale:latest            "sh -c /app/n9e"          31 minutes ago   Up 31 minutes   0.0.0.0:17000->17000/tcp, :::17000->17000/tcp                                                  nightingale
[root@aliyun-2c2g40g3m compose-bridge]# docker inspect 0500a886538e |grep Pid
            "Pid": 1619207,
            "PidMode": "",
            "PidsLimit": null,

```

2. 根据 Pid 进入 net namespace
==========================

1619207 就是 Pid，根据此 Pid 进入容器的 net namespace：

```
[root@aliyun-2c2g40g3m compose-bridge]# nsenter -t 1619207 -n bash
[root@aliyun-2c2g40g3m compose-bridge]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.22.0.6  netmask 255.255.0.0  broadcast 172.22.255.255
        ether 02:42:ac:16:00:06  txqueuelen 0  (Ethernet)
        RX packets 104463  bytes 33707078 (32.1 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 89615  bytes 10817199 (10.3 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 112  bytes 7096 (6.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 112  bytes 7096 (6.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

如上，使用 bash 命令进入 net namespace，然后执行 ifconfig，看到 IP：172.22.0.6，显然这就是容器的 IP，说明 nsenter 达成所愿，之后在这个 bash session 内执行 curl、telnet 之类的，就相当于在容器里执行一样的效果。完事执行 exit 命令可以退出这个 net namespace。

补充知识
====

上面的 1619207 这个 Pid 其实就是容器的一号进程的 Pid，在宿主机上执行下面的命令可以看出：

```
[root@aliyun-2c2g40g3m compose-bridge]# ps aux|grep n9e
root     1619207  0.0  0.0   2576   888 ?        Ss   15:22   0:00 sh -c /app/n9e
root     1619327  0.4  4.0 741264 78720 ?        Sl   15:22   0:09 /app/n9e
root     1620612  0.0  0.0 221528   864 pts/0    S+   16:01   0:00 grep --color=auto n9e

```

夜莺这个容器，核心执行的命令是 `/app/n9e`，不过在 docker-compose.yaml 中 command 写的是：

```
sh -c "/app/n9e"

```

所以这个容器的一号进程就是 sh 了。1619207 和 1619327 这俩进程是在一个容器里的，也就是在一个 net namespace 中的，这俩 Pid 都可以用于 nsenter，效果是一样的。

补充知识 2
======

除了 nsenter，使用 ip netns exec 也可以达到类似的效果。通过 ip netns help 可以看到相关帮助信息。

```
[root@aliyun-2c2g40g3m compose-bridge]# ip netns help
Usage: ip netns list
 ip netns add NAME
 ip netns attach NAME PID
 ip netns set NAME NETNSID
 ip [-all] netns delete [NAME]
 ip netns identify [PID]
 ip netns pids NAME
 ip [-all] netns exec [NAME] cmd ...
 ip netns monitor
 ip netns list-id [target-nsid POSITIVE-INT] [nsid POSITIVE-INT]
NETNSID := auto | POSITIVE-INT

```

从上面的 help 信息可以看出，要执行 ip netns exec，需要知道 net namespace 的 NAME，如何找到容器的 net namespace name 呢？

实际上，在 Linux 中，每个进程只要知道其 Pid 了，根据 Pid 就可以找到 net namespace 了，比如上面的例子，我们知道进程的 Pid 是：1619207，其 net namespace 就是：`/proc/1619207/ns/net`。

但是，执行 ip netns list 却看不到，这是因为，ip netns list 是罗列的 `/var/run/netns/` 下面的内容，而容器的 net namespace 并未挂到这里，此时，我们只需要做个软链挂过去即可：

```
ln -s /proc/1619207/ns/net /var/run/netns/1619207

```

如果发现 /var/run/netns 目录不存在，使用 root 账号 mkdir 一下即可。之后，我们就可以使用 ip netns exec 了：

```
ip netns exec 1619207 ifconfig

```

看到的输出和 nsenter 方式看到的输出是一样的。

如上，希望这个小知识可以帮到大家。

enjoy :-)

本公众号主理人：秦晓辉，极客时间《运维监控系统实战笔记》作者，Open-Falcon、夜莺、Categraf、Cprobe 等开源项目的创始人，当前在创业，为客户提供可观测性相关的产品。如下是我们两款核心产品，欢迎访问我们的官网了解详情：

https://flashcat.cloud/

我们主要提供两款产品：

![](http://i0.hdslb.com/bfs/article/c5f2c7e54078241da5b4cd1f9ca1c075442531657.png)

欢迎加我好友，交流可观测性相关话题或了解我们的商业产品，如下是我的联系方式，加好友请备注您的公司、姓名、来意 🤝

![](http://i0.hdslb.com/bfs/article/f4d046df8180ef770c09c9e335ac7520442531657.png)

扩展阅读：

方法论：面向故障处理的可观测性体系建设 (https://mp.weixin.qq.com/s/erX7Nl3IhmTihXpeBvmzHQ)

小总结：从 CTO 视角来看：如何搭建运维 / SRE 能力 (https://mp.weixin.qq.com/s/nelQpi02jgVLfSJlXYujlw)

鄙人专栏：运维监控系统实战笔记 (https://mp.weixin.qq.com/s/W7-AfNmPI1RYgo79WoXyxQ)