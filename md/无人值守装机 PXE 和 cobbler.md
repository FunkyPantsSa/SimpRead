> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/whcc/articles/15673734.html#2-pxe%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%92%8C%E5%AE%A2%E6%88%B7%E6%9C%BA%E7%9A%84%E5%B7%A5%E4%BD%9C%E8%BF%87%E7%A8%8B)

无人值守装机 PXE 和 cobbler
====================

一、DHCP
======

[https://www.cnblogs.com/hgzero/p/13176629.html#1. DHCP 简介](https://www.cnblogs.com/hgzero/p/13176629.html#1.%20DHCP%E7%AE%80%E4%BB%8B)

DHCP 服务搭建：[https://www.cnblogs.com/george-guo/p/5919444.html](https://www.cnblogs.com/george-guo/p/5919444.html)

1. DHCP（DHCP 服务）[#](#1-dhcpdhcp服务)
----------------------------------

DHCP（Dynamic Host Configuration Protocol）动态主机配置协议是一个局域网的网络协议。DHCP 服务可以自动给局域网中的主机自动分配一个 IP 地址，子网掩码，网关，广播地址等。

DHCP 是`C/S：Client/Server`。有两个端口`67`和`68`。跑的是 UDP 协议。分别对应`DHCP Server`和`DHCP Client`。

*   服务端：67/UDP
*   客户端：68/UDP

### 1. DHCP 的作用[#](#1-dhcp的作用)

*   把一个主机接入 TCP/IP 网络，并为其配置一些网络参数
    *   IP/Netmask
    *   Gateway
    *   DNS Server
*   参数的配置方式：
    *   静态指定
    *   动态分配
        *   bootp：boot protocol 引导协议，早期时候使用
        *   dhcp：引入了 “租约” 的 bootp，也可以实现为特定主机保留其固定地址

### 2. DHCP 三种分配方式[#](#2-dhcp三种分配方式)

*   自动分配：DHCP Server 给主机分配一个永久性的 IP 地址。
*   动态分配：DHCP Server 给主机分配一个具有时间限制的 IP 地址，到期地址可能会被其他主机租用。
*   手工分配：手工给主机分配一个 IP 地址。

### 3. DHCP 简要的工作流程[#](#3-dhcp简要的工作流程)

1.  要通信的主机要先在网络中广播发送一个`rarp`包（类似于客户主机广播说，我知道自己的 MAC 地址，谁提供一个 IP 地址给我）
    *   arp：address resolving protocol 地址解析协议
        *   IP --> MAC ，通过 IP 地址来找 MAC 地址
    *   **rarp：reverse arp 反向地址解析协议**
        *   MAC --> IP ，通过 MAC 地址来找 IP 地址
2.  服务器端（DHCP 服务器）收到请求之后，就会看一看，自己有没有地址，如果有，就提供一个地址给客户
    *   如果网络内有两台服务器都能为客户机提供 IP 地址，则谁响应快就由谁来提供

### 4. DHCP 工作的四个阶段：[#](#4-dhcp工作的四个阶段)

这些过程都是在同一个网络中使用**广播**的形式发送报文的，如果 DHCP 在另一个物理网路中，就要用到中继功能

**1) Client：发一个 dhcp discover 报文**

*   客户端发送请求报文，查找网络是否有 DHCP 服务器

**2) Server：回应一个 dhcp offer 报文（主要提供 IP/mask，gw 等信息）**

*   服务器收到报文后，发送一个 offer 报文给客户端主机
*   least time 租约期限，服务端还会发一个租约期限给客户端

**3) Client：再发送一个 dhcp request 报文**

*   因为可能网络中有多个 DHCP 服务器，这时 Client 发送的这个报文就是明确告知自己选择哪个 IP
*   客户端来确认到底使用哪一个 DHCP 服务提供的地址

**4) Server：回应一个 dhcp ack 报文**

*   DHCP 服务器来确认

总结：

*   DHCP Client 广播发送 Discover 报文 DHCP Server 发现
    
*   DHCP Server 响应发送 Offer 报文 DHCP Client 响应
    
*   DHCP Client 广播发送 Request 报文 DHCP Server 请求
    
*   DHCP Server 响应发送 ACK 报文 DHCP Client 确认
    

注：一般安装一个系统都会集成 DHCP Client 包的，所以一般无需安装。只需要在网卡配置从 DHCP 自动获取即可。

### 5. DHCP 的续租详解[#](#5--dhcp的续租详解)

*   续租：一般来说，租约期限到达整个期限一半的时候就会自动续组
    
    *   50% -> 75% -> 87.5% --> 93.75%
    *   四次续租不成功，就要重新请求了
*   **续租是单播给服务器的：**
    
    *   服务器端确认可以续租：
        *   `dhcp request`
        *   `dhcp ack` 同意续租；
*   服务器端确认不可以继续使用：（可能是地址列表给改了）
    
    *   dhcp request
    *   dhcp nak 不同意续租
    *   dhcp discover ：再以广播的形式发送探测报文
*   当客户端主机请求 DHCP 服务器请求失败时，自动随机分配的地址，这个地址只能在本地网络中使用
    
    *   169.254.X.X

实现 dhcp 对两个不同的局域网之间（两个局域网用路由器隔开）的地址分配：

*   可以在其中一个局域网中的一台主机上做中继（其实就是类似于代理的功能），然后将报文单播给另一个网络中的 DHCP 服务器
*   也可以在路由器中开启 dhcp 的中继功能（一般路由器应该有提供）
*   最好在两个网路中各有一个 DHCP 服务器。

### 6. dhcp 的主程序文件：[#](#6-dhcp的主程序文件)

程序包

*   dhcp：（由 ISC 提供的，named 用于 dns 服务）整合起来了
*   dnsmasq：既能提供 dhcp 又能提供 dns（通常用于嵌入式环境中，小规模使用，云计算）

（dhcpd 和 dhcrelay 只会启用其中的一个）

**1) /usr/sbin/dhcpd：提供 dhcp 服务**

*   配置文件：
    *   **/etc/dhcp/dhcpd.conf** --> /etc/rc.d/init.d/dhcpd
    *   有一个默认配置示例，可以复制过来：
        *   cp /usr/share/doc/dhcp-4.2.5/dhcpd.conf.example /etc/dhcp/dhcpd.conf
    *   /etc/dhcp/dhcpd6.conf --> /etc/rc.d/init.d/dhcpd6 这里是 IPV6 的配置文件，忽略它

**2) /usr/sbin/dhcrelay：提供中继服务（一般不启用）**

*   配置文件：/etc/rc.d/init.d/dhcrelay

2. DHCP 配置详解[#](#2-dhcp配置详解)
----------------------------

```
# 定义域名和DNS服务器地址

option domain-name "hgzero.com";                      # 定义搜索域的域名

option domain-name-servers 172.18.0.1, 172.18.0.2;    # 定义dns服务器



# 定义租约期限

default-lease-time 600;       # 默认租约期限，单位为秒钟，可以定义为一天，43200

max-lease-time 7200;          # 最长租约期限，可以定义为86400，24小时



# 定义记录日志的设备

log-facility local7;          # 记录日志的facility



# 定义一个子网和地址池，range表示地址池的范围

subnet 172.18.0.0 netmask 255.255.0.0 {

        range 172.18.100.101 172.18.100.120;

}

        

# 定义一个地址池，range就表示地址池的范围

subnet 10.254.239.0 netmask 255.255.255.224 {

  range 10.254.239.10 10.254.239.20;

  option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;

}



# 再定义一个子网和地址池，并对其做详细设定

subnet 10.5.5.0 netmask 255.255.255.224 {

  range 10.5.5.26 10.5.5.30;                           # 指明地址池的范围

  option domain-name-servers ns1.internal.example.org; # 指定dns服务器地址，最多指定3个

  option domain-name "internal.example.org";           # 指明所属的域，叫搜索后缀

  option routers 10.5.5.1;                             # 指明默认网关

  option broadcast-address 10.5.5.31;                  # 指定广播地址

  default-lease-time 600;                              # 默认租约时间

  max-lease-time 7200;                                 # 最大租约时间

} 



# 指明一个主机名，这个主机名只是在DHCP服务器上用来区别不同主机的标识

# 这里指明的主机名是passacaglia

host passacaglia {

  hardware ethernet 0:0:c0:5d:bd:95;   # 指明这台主机的mac地址

  filename "vmunix.passacaglia";       # 指明引导文件名称，就是获得地址之后要加载一个文件

                                      # 类似于一种基于网络引导时使用的bootloader文件

  server-name "toccata.fugue.com";     # 到这里指定的那个主机上去找那个文件，这里应该指定IP地址

}



# 对特定的主机绑定绑定IP地址（MAC地址绑定）

host fantasia {

  hardware ethernet 08:00:07:26:c0:a5;  # 这里的mac是客户端的mac地址

  fixed-address 10.0.10.51;     # fixed-address表示固定一个主机的地址，这里应该填IP地址，不能使用地址池范围内的地址

}


```

3. 配置后进行测试[#](#3-配置后进行测试)
-------------------------

1.  客户端可以使用
    
    **dhclient -d**
    
    命令来测试 dhcp 服务器
    
    *   dhclient -d 表示运行在前台
    *   如果不加参数则表示运行在后台
2.  可以在 **/var/lib/dhcpd/dhcpd.leases** 文件中查看 dhcp 的地址租约记录
    

4. 其他的配置选项[#](#4-其他的配置选项)
-------------------------

*   filename：指明引导文件名称，类似于一种基于网络引导时使用的 bootloader 文件
*   next-server：指明引导文件所在的服务主机的 IP 地址
*   配置示例：
    *   filename "pxelinux.0";
    *   next-server 172.18.100.6;
*   next-server 所在的文件服务器一般是 tftp 服务器
    *   tftp：trivial ftp，通过 udp 协议提供服务

5. dhcp 启动出错解决[#](#5-dhcp启动出错解决)
--------------------------------

**1）报错内容**

```
Not configured to listen on any interfaces!


```

**2）解决**

```
vim /usr/lib/system/system/dhcpd.service

    # 修改ExecStart项，在最后面添加上dhcp所绑定的网卡接口

    ExecStart=/usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid ens37



# 然后重载

systemctl daemon-reload

# 重启dhcp

systemctl restart dhcpd


```

6. 搭建 DHCP 服务[#](#6-搭建dhcp服务)
-----------------------------

### 服务端：[#](#服务端)

```
一、安装dhcp软件包

# yum info dhcp

Loaded plugins: fastestmirror, langpacks

Loading mirror speeds from cached hostfile

 * base: ftp.sjtu.edu.cn

 * extras: mirrors.ustc.edu.cn

 * updates: mirrors.bfsu.edu.cn

Installed Packages

Name        : dhcp

Arch        : x86_64

Epoch       : 12

Version     : 4.2.5

Release     : 83.el7.centos.1

Size        : 1.4 M

Repo        : installed

From repo   : updates

Summary     : Dynamic host configuration protocol software

URL         : http://isc.org/products/DHCP/

License     : ISC

Description : DHCP (Dynamic Host Configuration Protocol) is a protocol which allows

            : individual devices on an IP network to get their own network

            : configuration information (IP address, subnetmask, broadcast address,

            : etc.) from a DHCP server. The overall purpose of DHCP is to make it

            : easier to administer a large network.

            : 

            : To use DHCP on your network, install a DHCP service (or relay agent),

            : and on clients run a DHCP client daemon.  The dhcp package provides

            : the ISC DHCP service and relay agent.



# yum -y install dhcp

# rpm -ql dhcp

# man dhcpd.conf





二、配置DHCP服务器的IP 地址

这边测试选择用的是仅主机网络，并且确保VMware中的仅主机网络中的DHCP 关闭

# cat /etc/sysconfig/network-scripts/ifcfg-ens33 

TYPE="Ethernet"

BOOTPROTO="node"



DEVICE="ens33"

ONBOOT="yes"

IPADDR="172.18.1.10"

NETMASK="255.255.255.0"



# service network restart



三、改主配置文件



# cp -f  /usr/share/doc/dhcp-4.2.5/dhcpd.conf.example /etc/dhcp/dhcpd.conf （拷贝配置文件



# vi /etc/dhcp/dhcpd.conf                                  （修改配置文件



default-lease-time 600;                                   #设置默认租聘时间，单位为秒

max-lease-time 7200;                                      #设置最大租聘时间，单位为秒

log-facility local7;      

#设置记录日志级别，可以在syslog中查看

subnet 10.5.5.0 netmask 255.255.255.224 {                 #设置网段和掩码

	range 10.5.5.26 10.5.5.30;                            #设置地址段

	option domain-name-servers ns1.internal.example.org;  #设置主备DNS

	option domain-name "internal.example.org";            #设置一个域名，搜索后缀

	option routers 10.5.5.1;                              #设置网关

	option broadcast-address 10.5.5.31;                   #设置广播地址

	default-lease-time 600;                            #设置租聘时间，单位为秒

	max-lease-time 7200;                              #设置最大租聘时间，单位为秒

}



host fantasia {                                         #给主机单独分配一个静态IP

	hardware ethernet 08:00:07:26:c0:a5;                           #设置网卡MAC

	fixed-address 10.5.5.2;                                    #设置静态IP

}



四、启动服务

# systemctl start dhcpd                                    （开启服务）




```

### 客户端：网卡配置[#](#客户端网卡配置)

```
# dhclient ens33                                        （让网卡从DHCP服务获取一个IP（临时生效，重启失效）



# vi /etc/sysconfig/network-scripts/ifcfg-ens33                    （修改网卡配置，让网卡从DHCP服务获取IP（永久生效）



NAME=ens33

TYPE=Ethernet

DEVICE=ens33

ONBOOT=yes

BOOTPROTO=dhcp



# /etc/init.d/network reload                                （重载网卡




```

### 测试：[#](#测试)

新添加一台关闭了 DHCP 的仅主机的网络 DHCP 客户机

```
"# 服务端配置

<root@dhcp-server ~># cat /etc/dhcp/dhcpd.conf |grep -v '^#'

# dhcpd.conf

#

# Sample configuration file for ISC dhcpd

# option definitions common to all supported networks...

option domain-name "example.org";

option domain-name-servers ns1.example.org, ns2.example.org;



default-lease-time 60;

max-lease-time 70;



# have to hack syslog.conf to complete the redirection).

log-facility local7;



subnet 172.18.1.0 netmask 255.255.255.0 {

  range 172.18.1.15 172.18.1.20;

}

<root@dhcp-server ~># service dhcpd restart

# 服务端开启67 端口

<root@dhcp-server ~>#  ss -naltup | grep dhcpd

udp    UNCONN     0      0         *:67    *:*      users:(("dhcpd",pid=57548,fd=7))







" 客户端 测试从服务端获取IP

# 客户端开启68 号端口（动态获取IP 的进程）

<root@localhost ~># ss -naltup |grep 68



udp    UNCONN     0      0         *:68       *:*        users:(("dhclient",pid=81981,fd=6))



<root@localhost ~># service network restart

<root@localhost ~># dhclient -d

Internet Systems Consortium DHCP Client 4.2.5

Copyright 2004-2013 Internet Systems Consortium.

All rights reserved.



Listening on LPF/ens33/00:0c:29:d9:99:c2

Sending on   LPF/ens33/00:0c:29:d9:99:c2

Sending on   Socket/fallback

# 客户端向ens33网卡发送广播

# 客户端发送请求报文，查找网络是否有DHCP服务器

DHCPDISCOVER on ens33 to 255.255.255.255 port 67 interval 5 (xid=0x5d6b734d)

DHCPREQUEST on ens33 to 255.255.255.255 port 67 (xid=0x5d6b734d)

# DHCP服务端收到请求，并发送offer报文（主要提供IP/mask，gw等信息）给客户端

DHCPOFFER from 172.18.1.10

# 

DHCPACK from 172.18.1.10 (xid=0x5d6b734d)

# 服务从地址池分配一个租约期限内IP给客户端

bound to 172.18.1.15-- renewal in 286 seconds.





# 当租约时间每到续约时间的一半的时候就会向dhcp 服务端发起续约的请求

No DHCPOFFERS received.

No working leases in persistent database - sleeping.

No DHCPOFFERS received.

No working leases in persistent database - sleeping.

DHCPREQUEST on ens33 to 172.18.1.10 port 67 (xid=0x4b18a734)

DHCPACK from 172.18.1.10 (xid=0x4b18a734)

bound to 172.18.1.15 -- renewal in 34 seconds.



<root@localhost ~># ifconfig ens33

ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500

		# 从服务器获得的IP

        inet 172.18.1.15  netmask 255.255.255.0  broadcast 172.18.1.255



" 客户端 测试从服务端获取DNS和域名（搜索后缀）

# 服务端配置修改

<root@dhcp-server ~># cat /etc/dhcp/dhcpd.conf |grep -v '^#'

subnet 172.18.1.0 netmask 255.255.255.0 {

  range 172.18.1.15 172.18.1.20;

  option domain-name-servers 172.18.1.2;  # DNS

  option domain-name "abc.org";  # 域名

  option routers 172.18.1.1;   # 网关

}



<root@localhost ~># service network restart

<root@localhost ~># ifconfig ens33

ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500

        inet 172.18.1.15  netmask 255.255.255.0  broadcast 172.18.1.255

<root@localhost ~># cat /etc/resolv.conf 

# Generated by NetworkManager

search abc.org

nameserver 172.18.1.2



# 如果想使用自己的DNS地址，不希望使用服务器端给我们指定的DNS地址，

# 可以在客户端网卡配置文件添加  PEERDNS=no

<root@localhost ~># cat /etc/sysconfig/network-scripts/ifcfg-ens33 

PEERDNS=no



<root@localhost ~># route -n

Kernel IP routing table

Destination     Gateway         Genmask         Flags Metric Ref    Use Iface

0.0.0.0         172.18.1.1      0.0.0.0         UG    100    0        0 ens33

172.18.1.0      0.0.0.0         255.255.255.0   U     0      0        0 ens33



" 客户端 dhcp服务端给特定主机绑定IP地址





<root@dhcp-server ~># cat /etc/dhcp/dhcpd.conf |grep -v '^#'

# 对特定的主机绑定绑定IP地址（MAC地址绑定）

host fantasia {

  # 这里的mac是客户端的mac地址

  hardware ethernet 08:00:07:26:c0:a5;  

  # fixed-address表示固定一个主机的地址，这里应该填IP地址，不能使用地址池范围内的地址

  fixed-address 172.18.1.188;     

  option routers 172.18.1.10;   # 网关

}

<root@localhost ~># service network restart

<root@localhost ~># ifconfig ens33

ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500

        inet 172.18.1.188  netmask 255.255.255.0  broadcast 172.18.1.255

        inet6 fe80::51cd:a9c:d6a3:a9ef  prefixlen 64  scopeid 0x20<link>

        ether 00:0c:29:d9:99:c2  txqueuelen 1000  (Ethernet)

<root@localhost ~># route -n

Kernel IP routing table

Destination     Gateway         Genmask         Flags Metric Ref    Use Iface

0.0.0.0         172.18.1.10     0.0.0.0         UG    100    0        0 ens33

172.18.1.0      0.0.0.0         255.255.255.0   U     100    0        0 ens33




```

### 查看 dhcp-server 端分配的 IP：[#](#查看dhcp-server端分配的ip)

```
<root@dhcp-server ~># cat /var/lib/dhcpd/dhcpd.leases

# The format of this file is documented in the dhcpd.leases(5) manual page.

# This lease file was written by isc-dhcp-4.2.5



lease 172.18.1.15 {

  starts 3 2021/12/08 10:38:12;

  ends 3 2021/12/08 10:39:22;

  tstp 3 2021/12/08 10:39:22;

  cltt 3 2021/12/08 10:38:12;

  binding state active;

  next binding state free;

  rewind binding state free;

  hardware ethernet 00:0c:29:d9:99:c2;

}




```

```
centos6

[root@localhost ~]# cat anaconda-ks.cfg

# Kickstart file automatically generated by anaconda.



#version=DEVEL

install

cdrom

lang zh_CN.UTF-8

keyboard us

network --onboot no --device eth0 --bootproto dhcp --noipv6

rootpw  --iscrypted $6$gzE.KZif6zblJIMv$1zDMGa05lycCQ6d04LVg1S15f6bCa3awZDbD1BqxNUu.Z2UGQZCDJc.s/gYS3Vz10cEPpCDuKl5fS723X/8Nd.

firewall --service=ssh

authconfig --enableshadow --passalgo=sha512

selinux --enforcing

timezone Asia/Shanghai

bootloader --location=mbr --driveorder=sda --append="nomodeset crashkernel=auto rhgb quiet"

# The following is the partition information you requested

# Note that any partitions you deleted are not expressed

# here so unless you clear all partitions first, this is

# not guaranteed to work

#clearpart --linux --drives=sda

#volgroup VolGroup --pesize=4096 pv.008002

#logvol / --fstype=ext4 --name=lv_root --vgname=VolGroup --grow --size=1024 --maxsize=51200

#logvol swap --name=lv_swap --vgname=VolGroup --grow --size=2048 --maxsize=2048



#part /boot --fstype=ext4 --size=500

#part pv.008002 --grow --size=1





repo --  --baseurl=cdrom:sr0 --cost=100



%packages

@chinese-support

@core

@server-policy

@workstation-policy




```

7. DHCP 上的一些 PXE 的配置[#](#7-dhcp上的一些pxe的配置)
------------------------------------------

### dhcp 配置文件中的参数[#](#dhcp配置文件中的参数)

*   `filename`：指明引导文件名称，类似于一种基于网络引导时使用的`bootloader`引导加载程序文件
*   `next-server`：指明引导文件所在的服务主机的 IP 地址

### 配置示例[#](#配置示例)

*   man dhcpd.conf
    
*   `filename "pxelinux.0"`;
    
    *   一般都会提供这个文件，这个文件是 pxe 自己提供的
*   `next-server 172.18.100.6`;
    
    *   `next-server`指向的就是 tftp 服务器的地址
*   注意：文件是相对于 tftp 服务器的根路径的
    

### 关于 tftp[#](#关于tftp)

*   `next-server`所在的文件服务器一般是 tftp 服务器
*   `tftp`：`trivial ftp`，通过`udp`协议提供服务
*   监听端口：`69/udp`

二、PXE
=====

1. 介绍[#](#1-介绍)
---------------

*   `PXE`是`intel`公司推出的一款`通过网络来引导操作系统的协议`。`PXE`（`preboot execute environment`预启动执行环境）。让任何一台没有安装操作系统的主机，能够完成基于网络引导安装及启动操作。协议分为`client和server`两端，`pex client` 在网卡的 ROM 中，当计算机引导时，`BIOS`把`client`调入内存执行，并显示出安装命令菜单，经过用户选择后，`pxe client` 将放置在远端的操作系统通过网络下载到本地。
    
*   通过网络接口启动计算机，不依赖本地存储设备（如硬盘）或本地已安装的操作系统；
    
*   Client/Server 的工作模式
    
*   PXE 客户端会调用网际协议 (IP)、用户数据报协议(UDP)、动态主机设定协议(DHCP)、小型文件传输协议(TFTP) 等网络协议；
    
*   PXE 客户端 (client) 这个术语是指机器在 PXE 启动过程中的角色。一个 PXE 客户端可以是一台服务器、笔记本电脑
    

`PXE`服务器需要的支撑软件：`DHCP`、`TFTP`、`syslinux`（提供`pxe`引导程序文件`pxelinux.0`）、文件共享（`nfs、ftp、http、samba`）等。

**前提：**

*   客户端主机启动时，网卡必须支持网络引导机制
*   要将网卡调整为第一引导启动设备
*   客户机内存要大于或等于 2G

前提条件是计算机的网卡必须具有引导功能，这个网卡中要有一个 PXE 客户端。当计算机 POST 自检成功以后，BIOS 把网卡中 ROM 的 PXE 客户端调入内存执行，PXE 客户端通过网络中的 DHCP 服务器获取一个 IP 地址，拿到 IP 地址以后 PXE 继续引导计算机与网络中的 TFTP 客户端建立连接，从而从 TFTP 服务器中获取开机引导文件之后请求并下载安装需要的文件。在这个过程中需要一台服务器来提供启动文件、安装文件、以及安装过程中的自动应答文件等

2. PXE 服务器和客户机的工作过程：[#](#2-pxe服务器和客户机的工作过程)
-------------------------------------------

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211208015731209.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211208015731209.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211208015731209.png)

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211207223250601.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211207223250601.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211207223250601.png)

*   主机被唤醒后，开始加载网络引导应用时， pxe 客户端向 DHCP 服务器发送广播请求，网卡会通过在本地局域网中广播一条`rarp`协议的 UDP 包从`dhcp`服务器那里获得一个`IP`地址；
*   获得`IP`地址后，还会从`DHCP`那里获得一个`文件名`和一个`文件服务器地址`，随后，它会去找那个`tftp`文件服务器，去加载那个对应的文件；
*   等所需要的文件加载完成后，其会被在内存中展开，而后基于此文件可以去加载一个内核文件，此内核文件也一样会通过这个`tftp`文件服务器获取；加载内核通常还需要`initrd`虚根来完成对真实根所在的设备的驱动的加载，这一切操作都要通过这个`tftp`文件服务器来实现；
*   等加载完成后，这个内核文件通常是专用于为系统安装所设定的，因此，这个内核文件还需要去基于网络（通过配置在内核上的 IP 地址）把自己扮演成某种协议的客户端去加载一个能够启动安装程序的程序包（之前获得的 IP 地址是网卡上的 IP 地址，而不是内核上的 IP 地址）；
*   而后在本地完成安装并启动那个应用程序，而此程序已经不在这个 tftp 服务器上了；通常是一个基于 tfp 或者 http 获取 nfs 等服务所提供的一个 yum 仓库，通过这个 yum 仓库来加载安装程序，以及这个这个安装程序启动以后，很可能要读取 kickstart 文件，并根据 kickstart 文件的内容解决依赖关系后，基于这个 yum 仓库完成后续的所有安装过程。

1.  `PXE`客户机启动系统，选择从网卡启动，开始加载网络引导应用时。`pxe`客户端向`DHCP`服务器发送`rarp`协议的广播请求，该广播包为 UDP，请求`IP`地址信息
2.  ​ `DHCP`服务器响应`PXE`客户机的请求，自动从`IP`地址池中分配一个`IP`地址给`PXE`客户机，并且告知`PXE`客户机：`TFTP（`简单文件传输协议）服务器的`IP`地址和`PXE`引导程序文件`pxelinux.0`的位置。
3.  `pxe`客户端向`TFTP`服务器发送`pxelinux.0`文件的请求信息
4.  `TFTP`服务器发回给`PXE`客户端`pxelinux.0`文件相关信息
5.  `pxe`客户端执行引导程序文件`pxelinux.0`，读取到内存中，在内存中执行引导程序，并请求针对本机的 引导程序的配置信息文件`pxelinux.cfg/default`
6.  `TFTP`服务器发送`pxelinux.cfg/default`配置文件到`pxe`客户端
7.  `pxe`客户接收到`pxelinux.cfg/default`配置文件后读取它，并再次向`TFTP`请求`vmlinuz`和`initrd.img`
8.  `TFTP`响应请求返回`pxe`客户端`vmlimuz`内核文件和`initrd.img`
9.  `pxe`客户端读取`Linux`内核，然后启动安装程序
10.  Client 下载安装源文件，读取自动化安装脚本

最后安装系统时可以采用`kickstart`或者`vim`工具创建一个系统自动安装的应答文件，并用文件共享服务来共享`ks.cfg`文件（`ks.cfg`自动应答文件是记录系统安装的操作步骤，客户机在进行安装时会自动根据`ks.cfg`文件中的内容来完成安装操作）实现自动安装操作

3. PXE 中各服务器软件的功能[#](#3-pxe中各服务器软件的功能)
--------------------------------------

`DHCP`:

*   用来给`PXE`客户机自动分配`TCP/IP`设置（包括`IP地址、子网掩码、网关、DNS`等）。
*   告知`PXE`客户机`TFTP`服务器的`IP`地址和`PXE`启动文件名。
*   `filename`、`next-server`

`TFTP`：

*   是一个迷你的`FTP`共享协议软件，用来给`PXE`客户机提供所需的配置等文件。
*   TFTP 根目录`/var/lib/tftpboot/`
    *   `pxelinux.cfg/default`：安装菜单文件。通过这个 default 文件告诉客户机从什么内核引导, 以及在引导时向内核传递的任何选项
    *   `pxelinux.0`：给`PXE`客户机提供网络启动的引导程序文件, 生成安装界面和操作系统界面
    *   `vmlinuz`：“vm” 代表 “Virtual Memory”，是 Linux 是可引导的、压缩的内核镜像文件, 可以被引导程序加载，从而启动 Linux 系统。作用：进程管理、内存管理、文件管理、驱动管理、网络管理。
    *   `initrd.img`：是一个小的映象，包含一个最小的 linux 文件系统。通常的步骤是先启动内核，然后内核挂载`initrd.img`，并执行里面的脚本来进一步挂载各种各样的模块。其中最重要的就是根文件系统驱动模块，有了它才能挂载根文件系统，继而运行用户空间的第一个应用程序`init`或者`systemd`，完成系统后续的启动

`syslinux`：

*   `syslinux`是一个功能强大的引导加载程序，而且兼容各种介质。它的目的是简化首次安装 Linux 的时间，并建立修护或其它特殊用途的启动盘)。
    
*   用来提供`pxelinux.0`网络引导程序文件。
    

`文件共享`：

*   可以是`vsftpd、nfs、samba、http`等软件来实现文件共享。用来给`PXE`客户机提供系统安装文件。
*   `yum repository`：获取与加载的内核所匹配的发行版的 yum 仓库（可以通过 ftp、http、nfs 来提供）
*   一个`kickstart`文件，以完成全自动化的安装；若不需要全自动化安装，则可以不提供`kictstart`文件

4. 部署过程[#](#4-部署过程)
-------------------

参考：[https://blog.51cto.com/u_13588693/2355691](https://blog.51cto.com/u_13588693/2355691)

参考：[https://blog.csdn.net/weixin_40543283/article/details/84939014](https://blog.csdn.net/weixin_40543283/article/details/84939014)

参考： [https://www.cnblogs.com/qige2017/p/7545812.html](https://www.cnblogs.com/qige2017/p/7545812.html)

参考：[https://www.cnblogs.com/hgzero/p/13181922.html#5. Cobbler 的安装启动与配置](https://www.cnblogs.com/hgzero/p/13181922.html#5.%20Cobbler%E7%9A%84%E5%AE%89%E8%A3%85%E5%90%AF%E5%8A%A8%E4%B8%8E%E9%85%8D%E7%BD%AE)

部署过程详解：以下是在 centos7.6 系统里进行的操作，且 centos 系统 IP 为 192.168.11.11

部署前提检查 Linux 防火墙是否关闭和是否关闭 selinux 功能：

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211207225057294.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211207225057294.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211207225057294.png)

配置 DHCP 本地 yum 源

```
挂载光驱 ： 

cd /dev

mount /dev/sr0 /media

df -h 



vim /etc/yum.repos.d/dvd.media

[development]

name=rhel7

baseurl=file:///media

enabled=1

gpgcheck=0

yum clean all

yum makecache

yum -y install dhcp tftp-server

yum info dhcp



rpm -ql dhcp



<root@localhost dhcp># pwd

/etc/dhcp

<root@localhost dhcp># cp /usr/share/doc/dhcp-4.2.5/dhcpd.conf.example ./dhcpd.conf

cp: overwrite ‘./dhcpd.conf’? y

<root@localhost dhcp># ls

dhclient.d  dhclient-exit-hooks.d  dhcpd6.conf  dhcpd.conf   scripts



<root@localhost dhcp># grep -Ev '^#|^$' dhcpd.conf

# 全局的设置

option domain-name "example.org";

option domain-name-servers ns1.example.org, ns2.example.org;

default-lease-time 600;

max-lease-time 7200;

log-facility local7;

subnet 10.152.187.0 netmask 255.255.255.0 {

}

subnet 10.254.239.0 netmask 255.255.255.224 {

  range 10.254.239.10 10.254.239.20;

  option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;

}

subnet 10.254.239.32 netmask 255.255.255.224 {

  range dynamic-bootp 10.254.239.40 10.254.239.60;

  option broadcast-address 10.254.239.31;

  option routers rtr-239-32-1.example.org;

}



# 子网配置

subnet 192.168.19.0 netmask 255.255.255.0 {

  # 地址池的范围

  range 192.168.19.10 192.168.19.20;

  # DNS 服务器的地址（可以配置多个用 ， 隔开）

  option domain-name-servers 192.168.19.1, 192.168.19.3;

  # 域名（/etc/resolv.conf 中search exapmle.com）

  option domain-name "example.org";

  # 网关地址

  option routers 192.168.19.2;

  # 广播地址

  option broadcast-address 10.5.5.31;

  # 租约时间

  default-lease-time 600;

  max-lease-time 7200;

}



# 给某个主机固定IP地址

host passacaglia {

 

  hardware ethernet 0:0:c0:5d:bd:95;

  filename "vmunix.passacaglia";

  server-name "toccata.fugue.com";

}



host fantasia {

  # 请求IP地址的主机的MAC地址

  hardware ethernet 08:00:07:26:c0:a5;

  # 分配给固定主机的IP地址

  fixed-address xxxxx;

}

class "foo" {

  match if substring (option vendor-class-identifier, 0, 4) = "SUNW";

}

shared-network 224-29 {

  subnet 10.17.224.0 netmask 255.255.255.0 {

    option routers rtr-224.example.org;

  }

  subnet 10.0.29.0 netmask 255.255.255.0 {

    option routers rtr-29.example.org;

  }

  pool {

    allow members of "foo";

    range 10.17.224.10 10.17.224.250;

  }

  pool {

    deny members of "foo";

    range 10.0.29.10 10.0.29.230;

  }

}



<root@localhost dhcp># service dhcp configtest





<root@localhost dhcp># cat /etc/resolv.conf

# Generated by NetworkManager

search exapmle.com

nameserver 192.168.19.1

<root@localhost dhcp># ping server1

# ping server1会自动补齐为server1.exapmle.com

<root@localhost dhcp># ping server1.exapmle.com





<root@localhost dhcp># service dhcpd restart

<root@localhost dhcp># service dhcpd status

<root@localhost dhcp># netstat -naultp|grep :67

udp        0      0 0.0.0.0:67              0.0.0.0:*                           10474/dhcpd



<root@localhost dhcp># vim  /etc/selinux/config

<root@localhost dhcp># sed -ri 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

<root@localhost dhcp># setenforce 0

<root@localhost dhcp># getenforce

Permissive





service tftp

{

        socket_type             = dgram

        protocol                = udp

        wait                    = yes

        user                    = root

        server                  = /usr/sbin/in.tftpd

        server_args             = -s /var/lib/tftpboot

        disable                 = no   # yes 改为no

        per_source              = 11

        cps                     = 100 2

        flags                   = IPv4

}

<root@localhost dhcp># systemctl stop firewalld.service

<root@localhost dhcp># yum -y install xinetd




```

### 第一步：部署 DHCP 服务器[#](#第一步部署dhcp服务器)

关闭防火墙和 selinux

```
systemctl  stop  firewalld && systemctl  status  firewalld    

setenforce  0  &&  getenforce     


```

```
1.查软件是否已安装：

rpm  -q  dhcp

2.安装dhcp服务器软件：

yum   install  -y  dhcp

3.查配置文件列表：

rpm  -qc   dhcp

4.编辑dhcpd.conf配置文件

cat /etc/dhcp/dhcpd.conf

cp /usr/share/doc/dhcp-4.2.5/dhcpd.conf.example /etc/dhcp/dhcpd.conf



vim /etc/dhcp/dhcpd.conf   



# TFTP服务器的IP地址(可以写在全局) 

next-server 192.168.11.11;

# PXE引导程序文件名(可以写在全局) 默认是/pxelinux.0

filename "pxelinux.0";

# 定义网络地址和子网掩码

subnet 192.168.19.0 netmask 255.255.255.0{

  # 指定IP地址池的范围（起始和截止IP）

  range 192.168.19.10 192.168.19.20;

  # 域名解析服务器的IP地址

  option domain-name-servers 223.5.5.5;

  # 域名

  option domain-name "abc.org";

  # 网关IP地址

  option routers 192.168.19.2;

  # 广播地址

  option broadcast-address 12.168.19.255;

  # 默认租约时间

  default-lease-time 600;

  # 最大租约时间

  max-lease-time 7200;

}



5.启动dhcpd服务，允许服务开机自动启动



    systemctl     restart     dhcpd

    systemctl     enable      dhcpd

    systemctl     status      dhcpd

    netstat       -atunlp | grep    :67       查67号端口的网络进程序

   




```

### 第二步：部署 tftp-server 服务器[#](#第二步部署tftp-server服务器)

```
 1.查软件是否已安装：

 rpm  -q  tftp-server

 2.安装tftp-server服务器软件：

 yum info tftp-server

 yum   install  -y  tftp-server

 3.查配置文件列表：

 rpm  -qc   tftp-server

    /etc/xinetd.d/tftp

 4.编辑tftp配置文件

    vim   /etc/xinetd.d/tftp

  service tftp

{

        socket_type             = dgram

        protocol                = udp

        wait                    = yes

        user                    = root

        server                  = /usr/sbin/in.tftpd

        #tftp资源文件目录了，可以根据实际情况修改，一般不修改

        server_args             = -s /var/lib/tftpboot  

        disable                 = no   # yes 改为no

        per_source              = 11

        cps                     = 100 2

        flags                   = IPv4

}

 5.启动tftp服务，允许服务开机自动启动  

    #到这里如果提示错误，需要安装xinetd服务，一般安装tftp会自己安装没安装就手动安装下

    yum -y install xinetd

    systemctl restart xinetd

    systemctl enable xinetd

    systemctl status xinetd

    

    systemctl      restart     tftp

    systemctl       enable     tftp

    systemctl       status      tftp

    netstat       -atulp | grep    tftp

    netstat       -atunlp | grep    :69





   

# CentOS6上：

chkconfig tftp on

service xinetd restart



# CentOS7上

systemctl start tftp.socket


```

### 第三步：安装 syslinux 获取 pxelinux.0[#](#第三步安装syslinux-获取pxelinux0)

```
    

 1.安装提供pxelinux.0的syslinux软件，共享pxe引导程序文件

    rpm  -q  syslinux  mlocate

    yum  install  -y  syslinux  mlocate           安装指定的软件

    updatedb                                      更新locate文件查找数据库

    locate  pxelinux.0                             查找pxelinux.0文件

    rpm -ql syslinux   

       

 2.共享指定的文件到/var/lib/tftpboot 目录中 (这是单个系统，不用做，直接做第五步)

    cd   /var/lib/tftpboot                切换到tftp-server的默认共享目录

    cp  -v  /usr/share/syslinux/pxelinux.0   ./                复制指定的文件到当前目录中

    df  -hT                       查看磁盘空间使用状态

    cp  -v  /dvd/isolinux/*    ./                 复制光盘挂载点目录中指定的文件到当前目录中

    mkdir  -v  pxelinux.cfg                在当前目录中创建pxelinux.cfg目录

    cp  -v  isolinux.cfg   pxelinux.cfg/default        复制指定的文件到指定目录中并改名为default


```

### 第四步：配置 HTTP 服务[#](#第四步配置http服务)

```
 1. 安装httpd

 rpm -q httpd

 yum install httpd -y

 rpm -qc httpd

 

 mkdir -p /var/www/html/CentOS{6.9,7.9,8.5}

 

 2. 挂载光盘到目录

 lsblk

 mkdir /media/centos{6,7,8}

 mount /dev/sr0 /var/www/html/CentOS6

 mount /dev/sr1 /var/www/html/CentOS7

 mount /dev/sr2 /var/www/html/CentOS8

 df -hT

 

 可以选择开机自动挂载

 进入到配置文件中进行设置

  使用'vim /etc/fstab'进入/etc/fstab文件。

	/dev/cdrom　　　　 /mnt　　 iso9660　　 defaults　　 0 0

  mount -a 



 

 3. 启动httpd 服务并开启目录列表

 systemctl enable --now httpd.service 

 systemctl status httpd.service 

 

 # 删除文件 /etc/httpd/conf.d/welcome.conf

 vim /etc/httpd/conf/httpd.conf

     <Directory "/var/www/html">

        Options Indexes 

     </Directory>

     

 systemctl restart httpd




```

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211209012407689.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211209012407689.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211209012407689.png)

### 第五步：准备网络引导所需的文件[#](#第五步准备网络引导所需的文件)

准备网络引导时所需的`引导程序文件 pxelinux.0`，`安装菜单文件pxelinux.cfg/default`,`启动内核(vmlinuz)`和`影像文件(initrd.img)`，伪根系统等

```
cd   /var/lib/tftpboot

cp  -v  /usr/share/syslinux/pxelinux.0   /var/lib/tftpboot

mkdir /var/lib/tftpboot/centos{6,7,8}

cp /var/www/html/CentOS6/isolinux/* /var/lib/tftpboot/centos6

cp /var/www/html/CentOS7/isolinux/* /var/lib/tftpboot/centos7

cp /var/www/html/CentOS8/isolinux/* /var/lib/tftpboot/centos8

或者

cp /var/www/html/CentOSx/images/pxeboot/{vmlinuz,initrd.img} /var/lib/tftpboot/centosx



mkdir  -v  pxelinux.cfg 

cp  -v  /var/www/html/CentOS7/isolinux/isolinux.cfg   pxelinux.cfg/default     

chmod 644 pxelinux.cfg/default

cp  -v  /var/lib/tftpboot/centos7/vesamenu.c32  /var/lib/tftpboot/   

<root@localhost tftpboot># tree .

.

├── centos6

│   ├── boot.cat

│   ├── boot.msg

│   ├── grub.conf

│   ├── initrd.img

│   ├── isolinux.bin

│   ├── isolinux.cfg

│   ├── memtest

│   ├── splash.jpg

│   ├── TRANS.TBL

│   ├── vesamenu.c32

│   └── vmlinuz

├── centos7

│   ├── boot.cat

│   ├── boot.msg

│   ├── grub.conf

│   ├── initrd.img

│   ├── isolinux.bin

│   ├── isolinux.cfg

│   ├── memtest

│   ├── splash.png

│   ├── TRANS.TBL

│   ├── vesamenu.c32

│   └── vmlinuz

├── centos8

│   ├── boot.cat

│   ├── boot.msg

│   ├── grub.conf

│   ├── initrd.img

│   ├── isolinux.bin

│   ├── isolinux.cfg

│   ├── ldlinux.c32

│   ├── libcom32.c32

│   ├── libutil.c32

│   ├── memtest

│   ├── splash.png

│   ├── TRANS.TBL

│   ├── vesamenu.c32

│   └── vmlinuz

├── pxelinux.0            # 引导程序文件

└── pxelinux.cfg 

│   └── default           # 启动菜单配置文件

└── vesamenu.c32          # 图形背景




```

### 第六步：安装多版本 centos[#](#第六步安装多版本centos)

#### 方法一：[#](#方法一)

```
# 准备引导所需的配置文件

<root@localhost tftpboot># tree /var/lib/tftpboot/

/var/lib/tftpboot/

├── centos6

│   ├── boot.cat

│   ├── boot.msg

│   ├── grub.conf

│   ├── initrd.img

│   ├── isolinux.bin

│   ├── isolinux.cfg

│   ├── memtest

│   ├── pxelinux.0              # 引导程序文件

│   ├── pxelinux.cfg            # 启动菜单以及配置选项 

│   │   └── default

│   ├── splash.jpg

│   ├── TRANS.TBL

│   ├── vesamenu.c32

│   └── vmlinuz

├── centos7

│   ├── boot.cat

│   ├── boot.msg

│   ├── grub.conf

│   ├── initrd.img

│   ├── isolinux.bin

│   ├── isolinux.cfg

│   ├── memtest

│   ├── pxelinux.0 			# 引导程序文件

│   ├── pxelinux.cfg         # 启动菜单以及配置选项 

│   │   └── default

│   ├── splash.png

│   ├── TRANS.TBL

│   ├── vesamenu.c32

│   └── vmlinuz

├── centos8

│   ├── boot.cat

│   ├── boot.msg

│   ├── grub.conf

│   ├── initrd.img

│   ├── isolinux.bin

│   ├── isolinux.cfg

│   ├── ldlinux.c32

│   ├── libcom32.c32

│   ├── libutil.c32

│   ├── memtest

│   ├── pxelinux.0		    # 引导程序文件

│   ├── pxelinux.cfg         # 启动菜单以及配置选项 

│   │   └── default

│   ├── splash.png

│   ├── TRANS.TBL

│   ├── vesamenu.c32

│   └── vmlinuz





# 修改tftp 配置文件

<root@localhost tftpboot># vim   /etc/xinetd.d/tftp



service tftp

{

        socket_type             = dgram

        protocol                = udp

        wait                    = yes

        user                    = root

        server                  = /usr/sbin/in.tftpd

        # 修改server_args 根路径，想安装哪个版本，就改为那个目录就可以       

        server_args             = -s /var/lib/tftpboot/centos8

        disable                 = no

        per_source              = 11

        cps                     = 100 2

        flags                   = IPv4

}

<root@localhost tftpboot>#  systemctl restart xinetd




```

#### 方法二：[#](#方法二)

```
<root@localhost tftpboot># tree .

.

├── centos6

│   ├── boot.cat

│   ├── boot.msg

│   ├── grub.conf

│   ├── initrd.img

│   ├── isolinux.bin

│   ├── isolinux.cfg

│   ├── memtest

│   ├── splash.jpg

│   ├── TRANS.TBL

│   ├── vesamenu.c32

│   └── vmlinuz

├── centos7

│   ├── boot.cat

│   ├── boot.msg

│   ├── grub.conf

│   ├── initrd.img

│   ├── isolinux.bin

│   ├── isolinux.cfg

│   ├── memtest

│   ├── splash.png

│   ├── TRANS.TBL

│   ├── vesamenu.c32

│   └── vmlinuz

├── centos8

│   ├── boot.cat

│   ├── boot.msg

│   ├── grub.conf

│   ├── initrd.img

│   ├── isolinux.bin

│   ├── isolinux.cfg

│   ├── ldlinux.c32

│   ├── libcom32.c32

│   ├── libutil.c32

│   ├── memtest

│   ├── splash.png

│   ├── TRANS.TBL

│   ├── vesamenu.c32

│   └── vmlinuz        

├── pxelinux.0             # 引导程序文件

└── pxelinux.cfg           # 启动菜单以及配置选项的配置文件 

│    └── default           # default 来自centos8中的isolinux.cfg重命名（选择centos7中的也可以）

└── vesamenu.c32           # 来自centos7中的vesamenu.c32文件（用centos8中的有问题）






```

### 第六步：修改安装启动菜单来安装多版本 centos[#](#第六步修改安装启动菜单来安装多版本centos)

```
# 默认的菜单背景(如果default 选择的默认标签是linux7即default linux7 则不进入菜单图形界面，直接开始安装)

default vesamenu.c32          

timeout 600           # 菜单停留超时时间，如果想要显示图形菜单就把个值调小，比如timeout 60  6s



display boot.msg



# 菜单配色方案（不必要）

menu clear

menu background splash.png

menu title CentOS Linux 8

menu vshift 8

menu rows 18

menu margin 8

#menu hidden

menu helpmsgrow 15

menu tabmsgrow 13

# Border Area

menu color border * #00000000 #00000000 none

# Selected item

menu color sel 0 #ffffffff #00000000 none

# Title bar

menu color title 0 #ff7ba3d0 #00000000 none

# Press [Tab] message

menu color tabmsg 0 #ff3a6496 #00000000 none

# Unselected menu item

menu color unsel 0 #84b8ffff #00000000 none

# Selected hotkey

menu color hotsel 0 #84b8ffff #00000000 none

# Unselected hotkey

menu color hotkey 0 #ffffffff #00000000 none

# Help text

menu color help 0 #ffffffff #00000000 none

# A scrollbar of some type? Not sure.

menu color scrollbar 0 #ffffffff #ff355594 none

# Timeout msg

menu color timeout 0 #ffffffff #00000000 none

menu color timeout_msg 0 #ffffffff #00000000 none

# Command prompt text

menu color cmdmark 0 #84b8ffff #00000000 none

menu color cmdline 0 #ffffffff #00000000 none

# Do not display the actual menu unless the user presses a key. All that is displayed is a timeout message.

menu tabmsg Press Tab for full configuration options on menu items.

menu separator # insert an empty line

menu separator # insert an empty line



# 选项栏

label linux6.9

  menu label ^Install CentOS Linux 8

  

  kernel vmlinuz

  append initrd=initrd.img inst.stage2=hd:LABEL=CentOS-8-5-2111-x86_64-dvd quiet

label centos7.9

label centos8.5

label check

  menu label Test this ^media & install CentOS Linux 8

  # 进入菜单的高亮选行

  # 默认启动方式

  menu default       

  # 指定内核镜像文件位置，参考TFTP的根目录/var/lib/tftpboot的相对路径

  kernel vmlinuz     

  # 指定伪根系统所在位置同样参考TFTP服务器的根目录的相对路径，指定ks文件的位置。

  append initrd=initrd.img inst.stage2=hd:LABEL=CentOS-8-5-2111-x86_64-dvd rd.live.check quiet



menu separator # insert an empty line



# utilities submenu

menu begin ^Troubleshooting

  menu title Troubleshooting



label vesa

  menu indent count 5

  menu label Install CentOS Linux 8 in ^basic graphics mode

  text help

        Try this option out if you're having trouble installing

        CentOS Linux 8.

  endtext

  kernel vmlinuz

  append initrd=initrd.img inst.stage2=hd:LABEL=CentOS-8-5-2111-x86_64-dvd nomodeset quiet



label rescue

  menu indent count 5

  menu label ^Rescue a CentOS Linux system

  text help

        If the system will not boot, this lets you access files

        and edit config files to try to get it booting again.

  endtext

  kernel vmlinuz

  append initrd=initrd.img inst.stage2=hd:LABEL=CentOS-8-5-2111-x86_64-dvd inst.rescue quiet



label memtest

  menu label Run a ^memory test

  text help

        If your system is having issues, a problem with your

        system's memory may be the cause. Use this utility to

        see if the memory is working correctly.

  endtext

  kernel memtest



menu separator # insert an empty line



# 本地启动选项，在本实验中的测试机器是一台没有系统的干净虚拟机，所以不存在误删除本地系统的情况。要考虑PXE通过网络引导自动安装操作系统一键安装，而我们的安装策略又是格式化光盘，清除所有磁盘分区表。如果测试的机器是一台有系统的机器，就有可能误操作破坏原有的操作系统。所以增加一项本地磁盘启动的选项，并设为默认启动项，当菜单停留时间超时以后，自动选择该选项。

label local             # 从硬盘启动

  menu label Boot from ^local drive

  localboot 0xffff



menu separator # insert an empty line

menu separator # insert an empty line



label returntomain

  menu label Return to ^main menu

  menu exit



menu end




```

```
kickstart文件的格式



命令段：指明各种安装前配置，如键盘类型等

必备命令 ：

authconfig: 认证方式配置

authconfig --useshadow --passalgo=sha512

bootloader：bootloader的安装位置及相关配置

bootloader --location=mbr --driveorder=sda – append="crashkernel=auto rhgb quiet"

keyboard: 设定键盘类型

lang: 语言类型

part: 创建分区

rootpw: 指明root的密码

timezone: 时区

可选命令 ：

install OR upgrade

text: 文本安装界面

network

firewall

selinux

halt

poweroff

reboot

repo

user：安装完成后为系统创建新用户

url: 指明安装源 key –skip 跳过安装号码,适用于rhel版本



程序包段：指明要安装的程序包组或程序包，不安装的程序包等

%packages

@group_name

package

-package #如果要安装的包组中包含不想要的包用-号代表剔除

%end #指定要安装的程序包以%packages开头，%end 结尾

脚本段：

%pre: 安装前脚本

运行环境：运行于安装介质上的微型Linux环境

%post: 安装后脚本

运行环境：安装完成的系统







#version=DEVEL

# System authorization information

auth --enableshadow --passalgo=sha512

# Use CDROM installation media

cdrom

# Use graphical install

graphical

# Run the Setup Agent on first boot

firstboot --enable

ignoredisk --only-use=sda

# Keyboard layouts

keyboard --vckeymap=us --xlayouts='us'

# System language

lang en_US.UTF-8



# Network information

network  --bootproto=dhcp --device=ens33 --ipv6=auto --activate

network  --hostname=localhost.localdomain



# Root password

rootpw --iscrypted $6$U5OsGHHitXI9E98W$c7Bpu8i0V/5ZJqhkQi9MyJJPqkH1cDLUe3eW5un.tuU16hcw0M46Ld94zpG8Z3RZ4b9T664NX0dXy4za4QeXO1

# System services

services --enabled="chronyd"

# System timezone

timezone Asia/Shanghai --isUtc

user --

# System bootloader configuration

bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda

autopart --type=lvm

# Partition clearing information

clearpart --none --initlabel



%packages

@^minimal

@core

@development

chrony

kexec-tools



%end



%addon com_redhat_kdump --enable --reserve-mb='auto'



%end



%anaconda

pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty

pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok

pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty

%end









#platform=x86, AMD64, 或 Intel EM64T

#version=DEVEL

# Install OS instead of upgrade

install

# Keyboard layouts

keyboard 'us'

# System language

lang en_US

# System authorization information

auth  --useshadow  --passalgo=sha512

# Use graphical install

graphical

firstboot --disable

# SELinux configuration

selinux --disabled





# Firewall configuration

firewall --disabled

# Reboot after installation

reboot

# System timezone

timezone Asia/Shanghai

# Use network installation

url --url="http://192.168.19.100/CentOS7"

# System bootloader configuration

bootloader --location=mbr

# Clear the Master Boot Record

zerombr

# Partition clearing information

clearpart --all --initlabel

# Disk partitioning information

part /boot --fstype="xfs" --size=1024

part swap --fstype="swap" --size=4096

part / --fstype="xfs" --grow --size=1



%post

#!/bin/bash

yum -y install httpd

echo 1111 > root.txt

%end



%packages

@additional-devel

@development

@fonts

@ftp-server

@gnome-desktop

@ha

@java-platform

@mariadb

@platform-devel

@remote-system-management

@system-management

@virtualization-client

@virtualization-hypervisor

@virtualization-platform

@virtualization-tools

fence-agents-all

-cjkuni-uming-fonts

-dejavu-sans-fonts

-dejavu-sans-mono-fonts

-dejavu-serif-fonts

-gnu-free-mono-fonts

-gnu-free-sans-fonts

-gnu-free-serif-fonts

-google-crosextra-caladea-fonts

-google-crosextra-carlito-fonts

-google-noto-emoji-fonts

-jomolhari-fonts

-khmeros-base-fonts

-liberation-mono-fonts

-liberation-sans-fonts

-liberation-serif-fonts

-lklug-fonts

-lohit-assamese-fonts

-lohit-bengali-fonts

-lohit-devanagari-fonts

-lohit-gujarati-fonts

-lohit-kannada-fonts

-lohit-malayalam-fonts

-lohit-marathi-fonts

-lohit-nepali-fonts

-lohit-oriya-fonts

-lohit-punjabi-fonts

-lohit-tamil-fonts

-lohit-telugu-fonts

-madan-fonts

-nhn-nanum-gothic-fonts

-open-sans-fonts

-overpass-fonts

-paktype-naskh-basic-fonts

-paratype-pt-sans-fonts

-qgnomeplatform

-sil-abyssinica-fonts

-sil-nuosu-fonts

-sil-padauk-fonts

-smc-meera-fonts

-stix-fonts

-thai-scalable-waree-fonts

-ucs-miscfixed-fonts

-vlgothic-fonts

-wqy-microhei-fonts

-wqy-zenhei-fonts

-xdg-desktop-portal-gtk



%end







%packages

@base

@development

@remote-system-management

@system-management

-abrt-addon-ccpp

-abrt-addon-python

-abrt-cli

-abrt-console-notification

-bash-completion

-blktrace

-bpftool

-bridge-utils

-bzip2

-chrony

-cryptsetup

-dmraid

-dosfstools

-ethtool

-fprintd-pam

-gnupg2

-hunspell

-hunspell-en

-kmod-kvdo

-kpatch

-ledmon

-libaio

-libreport-plugin-mailx

-libstoragemgmt

-lvm2

-man-pages

-man-pages-overrides

-mdadm

-mlocate

-mtr

-nano

-ntpdate

-pinfo

-plymouth

-pm-utils

-rdate

-rfkill

-rng-tools

-rsync

-scl-utils

-setuptool

-smartmontools

-sos

-sssd-client

-strace

-sysstat

-systemtap-runtime

-tcpdump

-tcsh

-teamd

-time

-unzip

-usbutils

-vdo

-vim-enhanced

-virt-what

-wget

-which

-words

-xfsdump

-xz

-yum-langpacks

-yum-utils

-zip



%end







"centos6 

#platform=x86, AMD64, 或 Intel EM64T

#version=DEVEL

# Install OS instead of upgrade

install

# Keyboard layouts

keyboard 'us'

# Root password (111111)

rootpw --iscrypted $1$DaMGwMzL$.v/4oq8xO2g.rxXDVjft10

# System language

lang en_US

# System authorization information

auth  --useshadow  --passalgo=sha512

# Use graphical install

graphical

firstboot --disable

# SELinux configuration

selinux --disabled



# Firewall configuration

firewall --disabled

# Reboot after installation

reboot

# System timezone

timezone Asia/Shanghai

# Use network installation

url --url="http://192.168.19.100/CentOS6"

# System bootloader configuration

bootloader --location=mbr

# Clear the Master Boot Record

zerombr

# Partition clearing information

clearpart --all --initlabel

# Disk partitioning information

part /boot --fstype="ext4" --size=1024

part swap --fstype="swap" --size=4096

part / --fstype="ext4" --grow --size=1



%post

#!/bin/bash

yum -y install httpd

echo 1111 > root.txt

%end





%packages

@core

@server-policy

@workstation-policy

%end







"centos7 

#platform=x86, AMD64, 或 Intel EM64T

#version=DEVEL

# Install OS instead of upgrade

install

# Keyboard layouts

keyboard 'us'

# Root password (111111)

rootpw --iscrypted $1$DaMGwMzL$.v/4oq8xO2g.rxXDVjft10

# System language

lang en_US

# System authorization information

auth  --useshadow  --passalgo=sha512

# Use graphical install

graphical

firstboot --disable

# SELinux configuration

selinux --disabled



# Firewall configuration

firewall --disabled

# Reboot after installation

reboot

# System timezone

timezone Asia/Shanghai

# Use network installation

url --url="http://192.168.19.100/CentOS7"

# System bootloader configuration

bootloader --location=mbr

# Clear the Master Boot Record

zerombr

# Partition clearing information

clearpart --all --initlabel

# Disk partitioning information

part /boot --fstype="xfs" --size=1024

part swap --fstype="swap" --size=4096

part / --fstype="xfs" --grow --size=1



%post

#!/bin/bash

yum -y install httpd

echo 1111 > root.txt

%end



%packages

@^minimal

@core

@development

chrony

kexec-tools



%end





"centos8"

#platform=x86, AMD64, 或 Intel EM64T

#version=DEVEL

# Install OS instead of upgrade

install

# Keyboard layouts

keyboard 'us'

# Root password (111111)

rootpw --iscrypted $1$DaMGwMzL$.v/4oq8xO2g.rxXDVjft10

# System language

lang en_US

# System authorization information

auth  --useshadow  --passalgo=sha512

# Use graphical install

graphical

firstboot --disable

# SELinux configuration

selinux --disabled



# Firewall configuration

firewall --disabled

# Reboot after installation

reboot

# System timezone

timezone Asia/Shanghai

# Use network installation

url --url="http://192.168.19.100/CentOS7"

# System bootloader configuration

bootloader --location=mbr

# Clear the Master Boot Record

zerombr

# Partition clearing information

clearpart --all --initlabel

# Disk partitioning information

part /boot --fstype="xfs" --size=1024

part swap --fstype="swap" --size=4096

part / --fstype="xfs" --grow --size=1



%packages

@^minimal-environment

@legacy-unix

@standard

@system-tools



%end













"centos8"

<root@localhost ~># cat anaconda-ks.cfg

#version=RHEL8

# Use graphical install

graphical



repo -- --baseurl=file:///run/install/sources/mount-0000-cdrom/AppStream



%packages

@^minimal-environment

@legacy-unix

@standard

@system-tools



%end



# Keyboard layouts

keyboard --xlayouts='cn'

# System language

lang zh_CN.UTF-8



# Network information

network  --bootproto=dhcp --device=ens160 --ipv6=auto --activate

network  --hostname=localhost.localdomain



# Use CDROM installation media

cdrom



# Run the Setup Agent on first boot

firstboot --enable



ignoredisk --only-use=nvme0n1

autopart

# Partition clearing information

clearpart --none --initlabel



# System timezone

timezone Asia/Shanghai --isUtc



# Root password

rootpw --iscrypted $6$g9Qqs5dLZNPWPZnM$7DDmh6JyysdAyYLLqjkWlri1OFlHmSTh.ZTUjGvQTQSgbBFuaPUpg9s3eFbLzGdvJgLxTtdxaQKR9yH7Wmlpz/



%addon com_redhat_kdump --disable --reserve-mb='auto'



%end



%anaconda

pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty

pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok

pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty

%end



"centos7

<root@localhost isolinux># cat /root/anaconda-ks.cfg 

#version=DEVEL

# System authorization information

auth --enableshadow --passalgo=sha512

# Use CDROM installation media

cdrom

# Use graphical install

graphical

# Run the Setup Agent on first boot

firstboot --enable

ignoredisk --only-use=sda

# Keyboard layouts

keyboard --vckeymap=us --xlayouts='us'

# System language

lang en_US.UTF-8



# Network information

network  --bootproto=dhcp --device=ens33 --ipv6=auto --activate

network  --hostname=localhost.localdomain



# Root password

rootpw --iscrypted $6$U5OsGHHitXI9E98W$c7Bpu8i0V/5ZJqhkQi9MyJJPqkH1cDLUe3eW5un.tuU16hcw0M46Ld94zpG8Z3RZ4b9T664NX0dXy4za4QeXO1

# System services

services --enabled="chronyd"

# System timezone

timezone Asia/Shanghai --isUtc

user --

# System bootloader configuration

bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda

autopart --type=lvm

# Partition clearing information

clearpart --none --initlabel



%packages

@^minimal

@core

@development

chrony

kexec-tools



%end



%addon com_redhat_kdump --enable --reserve-mb='auto'



%end



%anaconda

pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty

pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok

pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty

%end



"centos6"



[root@localhost ~]# cat anaconda-ks.cfg

# Kickstart file automatically generated by anaconda.



#version=DEVEL

install

cdrom

lang en_US.UTF-8

keyboard us

network --onboot no --device eth0 --bootproto dhcp --noipv6

rootpw  --iscrypted $6$/JPzCCTjmsAax3E.$Ib.61okHko0kp7qCU7DZDyZzvzZ8lgHWOl36yI2Ltbafu/hoJPo70OYhzw2M3.65pvrIbh8ljcE87E9HKycvZ/

firewall --service=ssh

authconfig --enableshadow --passalgo=sha512

selinux --enforcing

timezone Asia/Shanghai

bootloader --location=mbr --driveorder=sda --append="crashkernel=auto rhgb quiet"

# The following is the partition information you requested

# Note that any partitions you deleted are not expressed

# here so unless you clear all partitions first, this is

# not guaranteed to work

#clearpart --none



#part /boot --fstype=ext4 --size=1000

#part swap --size=2048

#part / --fstype=ext4 --grow --size=200





repo -- --cost=100



%packages

@core

@server-policy

@workstation-policy

%end




```

### ks-centos6-cfg[#](#ks-centos6-cfg)

```
"最小化安装"

<root@localhost html># cat ks-centos6.cfg

#platform=x86, AMD64, 或 Intel EM64T

#version=DEVEL

# Install OS instead of upgrade

install

# Keyboard layouts

keyboard 'us'

# Root password (111111)

rootpw --iscrypted $1$DaMGwMzL$.v/4oq8xO2g.rxXDVjft10

# System language

lang en_US

# System authorization information

auth  --useshadow  --passalgo=sha512

# Use graphical install

graphical

firstboot --disable

# SELinux configuration

selinux --disabled



# Firewall configuration

firewall --disabled

# Reboot after installation

reboot

# System timezone

timezone Asia/Shanghai

# Use network installation

url --url="http://192.168.19.100/CentOS6"

# System bootloader configuration

bootloader --location=mbr

# Clear the Master Boot Record

zerombr

# Partition clearing information

clearpart --all --initlabel

# Disk partitioning information

part /boot --fstype="ext4" --size=1024

part swap --fstype="swap" --size=4096

part / --fstype="ext4" --grow --size=1



%post

#!/bin/bash

yum -y install httpd

echo 1111 > root.txt

%end





%packages

@core

@server-policy

@workstation-policy

%end



"最小化图形安装"

<root@localhost html># cat ks-centos6.cfg

#platform=x86, AMD64, 或 Intel EM64T

#version=DEVEL

# Install OS instead of upgrade

install

# Keyboard layouts

keyboard 'us'

# Root password (111111)

rootpw --iscrypted $1$DaMGwMzL$.v/4oq8xO2g.rxXDVjft10

# System language

lang en_US

# System authorization information

auth  --useshadow  --passalgo=sha512

# Use graphical install

graphical

firstboot --disable

# SELinux configuration

selinux --disabled



# Firewall configuration

firewall --disabled

# Reboot after installation

reboot

# System timezone

timezone Asia/Shanghai

# Use network installation

url --url="http://192.168.19.100/CentOS6"

# System bootloader configuration

bootloader --location=mbr

# Clear the Master Boot Record

zerombr

# Partition clearing information

clearpart --all --initlabel

# Disk partitioning information

part /boot --fstype="ext4" --size=1024

part swap --fstype="swap" --size=4096

part / --fstype="ext4" --grow --size=1



%post

#!/bin/bash

yum -y install httpd

echo 1111 > root.txt

%end



%packages

@base

@chinese-support

@core

@debugging

@basic-desktop

@desktop-debugging

@desktop-platform

@directory-client

@fonts

@input-methods

@internet-browser

@java-platform

@legacy-x

@network-file-system-client

@print-client

@remote-desktop-clients

@server-platform

@server-policy

@workstation-policy

@x11

mtools

pax

python-dmidecode

oddjob

sgpio

device-mapper-persistent-data

abrt-gui

samba-winbind

certmonger

pam_krb5

krb5-workstation

libXmu

%end[









@gnome-desktop

-qgnomeplatform

-xdg-desktop-portal-gtk




```

### ks-centos7.cfg[#](#ks-centos7cfg)

```
<root@localhost html># cat ks-centos7.cfg

#platform=x86, AMD64, 或 Intel EM64T

#version=DEVEL

# Install OS instead of upgrade

install

# Keyboard layouts

keyboard 'us'

# Root password

rootpw --iscrypted $1$DaMGwMzL$.v/4oq8xO2g.rxXDVjft10

# System language

lang en_US

# System authorization information

auth  --useshadow  --passalgo=sha512

# Use graphical install

graphical

firstboot --disable

# SELinux configuration

selinux --disabled





# Firewall configuration

firewall --disabled

# Reboot after installation

reboot

# System timezone

timezone Asia/Shanghai

# Use network installation

url --url="http://192.168.19.100/CentOS7"

# System bootloader configuration

bootloader --location=mbr

# Clear the Master Boot Record

zerombr

# Partition clearing information

clearpart --all --initlabel

autopart --type=lvm

# Disk partitioning information

#part /boot --fstype="xfs" --size=1024

#part swap --fstype="swap" --size=4096

#part / --fstype="xfs" --grow --size=1



%post

yum -y install httpd

echo 1111 > /root/root.txt

%end



%packages

@^minimal

@core

@development

chrony

kexec-tools



%end




```

### ks-centos8.cfg[#](#ks-centos8cfg)

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211210003922863.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211210003922863.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211210003922863.png)

```
<root@localhost html># cat ks-centos8.cfg

#version=RHEL8

# Reboot after installation

reboot

# Use graphical install

graphical



repo -- --baseurl=http://192.168.19.100/CentOS8/AppStream



%packages

@^minimal-environment

@legacy-unix

@standard

@system-tools

kexec-tools



%end



# Keyboard layouts

keyboard --vckeymap=us --xlayouts='us'

# System language

lang en_US.UTF-8

# Root password (111111)

rootpw --iscrypted $1$DaMGwMzL$.v/4oq8xO2g.rxXDVjft10

# Firewall configuration

firewall --disabled

# Network information

network  --hostname=localhost.localdomain



# Use network installation

url --url="http://192.168.19.100/CentOS8"



# System authorization information

auth --useshadow --passalgo=sha512

# SELinux configuration

selinux --disabled



firstboot --disable



ignoredisk --only-use=nvme0n1

# System bootloader configuration

bootloader --append="crashkernel=auto" --location=mbr --boot-drive=nvme0n1

# Clear the Master Boot Record

zerombr

# Partition clearing information

clearpart --all --initlabel

autopart --type=lvm

# Disk partitioning information

#part /date --fstype="xfs" --size=8888

#part /boot --fstype="xfs" --size=1024

#part swap --fstype="swap" --size=4096

#part / --fstype="xfs" --size=6470



# System timezone

timezone Asia/Shanghai



# Root password

rootpw --iscrypted $1$DaMGwMzL$.v/4oq8xO2g.rxXDVjft10



%addon com_redhat_kdump --enable --reserve-mb='auto'



%end





---------------------------------------------------------------------------



#version=RHEL8

# Reboot after installation

reboot

# Use graphical install

graphical



repo -- --baseurl=http://192.168.19.100/CentOS8/AppStream



%packages

@^minimal-environment

@legacy-unix

@standard

@system-tools

kexec-tools



%end



# Keyboard layouts

keyboard --vckeymap=us --xlayouts='us'

# System language

lang en_US.UTF-8



# Root password (111111)

rootpw --iscrypted $1$DaMGwMzL$.v/4oq8xO2g.rxXDVjft10



# Firewall configuration

firewall --disabled

# Network information

network  --hostname=localhost.localdomain



# Use network installation

url --url="http://192.168.19.100/CentOS8"



# System authorization information

auth --useshadow --passalgo=sha512

# SELinux configuration

selinux --disabled



firstboot --disable



ignoredisk --only-use=nvme0n1

# System bootloader configuration

bootloader --append="crashkernel=auto" --location=mbr --boot-drive=nvme0n1

# Clear the Master Boot Record

zerombr

# Partition clearing information

clearpart --all --initlabel

# Disk partitioning information

part /date --fstype="xfs" --size=8888

part /boot --fstype="xfs" --size=1024

part swap --fstype="swap" --size=4096

part / --fstype="xfs" --size=6470



# System timezone

timezone Asia/Shanghai



# Root password

rootpw --iscrypted $1$DaMGwMzL$.v/4oq8xO2g.rxXDVjft10



%addon com_redhat_kdump --enable --reserve-mb='auto'



%end





-------------------------------

centos8 桌面



[root@localhost ~]# vim anaconda-ks.cfg 

[root@localhost ~]# cat anaconda-ks.cfg 

#version=RHEL8

# Use graphical install

graphical



repo -- --baseurl=file:///run/install/sources/mount-0000-cdrom/AppStream



%packages

@^graphical-server-environment

kexec-tools



%end



# Keyboard layouts

keyboard --xlayouts='cn'

# System language

lang zh_CN.UTF-8



# Network information

network  --bootproto=dhcp --device=ens160 --ipv6=auto --activate

network  --hostname=localhost.localdomain



# Use CDROM installation media

cdrom



# Run the Setup Agent on first boot

firstboot --enable



ignoredisk --only-use=nvme0n1

# Partition clearing information

clearpart --none --initlabel

# Disk partitioning information

part pv.116 --fstype="lvmpv" --ondisk=nvme0n1 --size=19455

part /boot --fstype="xfs" --ondisk=nvme0n1 --size=800

volgroup cl --pesize=4096 pv.116

logvol swap --fstype="swap" --size=2047 --name=swap --vgname=cl

logvol / --fstype="xfs" --grow --size=1024 --name=root --vgname=cl



# System timezone

timezone Asia/Shanghai --isUtc --nontp



# Root password

rootpw --iscrypted $6$gqWBuBF7pUtDXQKo$ClukSa7KpW4hx0NwCFLgWCylk/BRmOTTwfr/9/J5ezM6hgzfLMY3npIT700ohAoQ3fnlx3scjlvQ7CbNjG24O.



%addon com_redhat_kdump --enable --reserve-mb='auto'



%end



%anaconda

pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty

pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok

pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty

%end




```

```
label linux6

  menu label Auto Install CentOS linux ^6

  #menu label ^Install CentOS Linux 6

  #menu default

  kernel /centos6/vmlinuz

  append initrd=/centos6/initrd.img ks=http://192.168.19.100/ks-centos6.cfg



label linux7

  menu label Auto Install CentOS linux ^7

  #menu label ^Install CentOS Linux 7

  #menu default

  kernel /centos7/vmlinuz

  # append:表示向内核传递的参数，安装所需的仓库地址

  append initrd=/centos7/initrd.img inst.repo=http://192.168.19.100/CentOS7 inst.ks=http://192.168.19.100/ks.cfg



label linux8

  menu label Auto Install CentOS linux ^8

  #menu label ^Install CentOS Linux 6

  menu default

  kernel /centos8/vmlinuz

  append initrd=/centos8/initrd.img inst.ks=http://192.168.19.100/ks-centos8.cfg






```

### 第七步：新建虚拟机测试[#](#第七步新建虚拟机测试)

新建一个和 DHCP 服务端在一个网络内的客户机，由于 DHCP 选择的是 net , 这儿也选择 net 网络。（一定记得关闭原来的 DHCP）

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211209023014554.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211209023014554.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211209023014554.png)

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211209024003716.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211209024003716.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211209024003716.png)

centos8 错误`vesamenu.c32:not a COM32R image`

解决办法：

```
cp /var/lib/tftpboot/centos7/vesamenu.c32 /var/lib/tftpboot/centos8/vesamenu.c32 


```

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211209032055119.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211209032055119.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211209032055119.png)

5. 总结[#](#5-总结)
---------------

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211209041024119.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211209041024119.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211209041024119.png)

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211210014954863.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211210014954863.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211210014954863.png)

### 第四步：部署文件共享服务器（以 vsftp 为例）[#](#第四步部署文件共享服务器以vsftp为例)

（可用的有 vsftpd、nfs、samba、httpd）

```
1.查软件是否已安装：rpm  -q  vsftpd

 2.安装tftp-server服务器软件：yum   install  -y  vsftpd

 3.查配置文件列表：rpm  -qc   vsftpd

 4.启动vsftpd服务，允许开机自动启动服务

    systemctl  restart  vsftpd

    systemctl  enable  vsftpd

    systemctl  status  vsftpd

    netstat  -atunlp | grep  :21   或  lsof  -i  :21

 5.共享centos7的系统镜像到/var/ftp/dvd目录

    mkdir  -v  /var/ftp/dvd

    mount  /dev/sr0   /var/ftp/dvd




```

三、kickstart 配置文件
================

参考：[https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/sect-simple-install-kickstart](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/sect-simple-install-kickstart)

参考：[https://blog.csdn.net/taiyang1987912/article/details/42176709](https://blog.csdn.net/taiyang1987912/article/details/42176709)

```
关键字     含义

install     告知安装程序，这是一次全新安装，而不是升级upgrade。

url --url=" "     通过FTP或HTTP从远程服务器上的安装树中安装。

url --url="http://10.0.10.11/CentOS7/"

url --url ftp://<username>:<password>@<server>/<dir>

nfs     从指定的NFS服务器安装。

nfs --server=nfsserver.example.com --dir=/tmp/install-tree

text     使用文本模式安装。

lang     设置在安装过程中使用的语言以及系统的缺省语言。lang en_US.UTF-8

keyboard     设置系统键盘类型。keyboard us

zerombr     清除mbr引导信息。

bootloader     系统引导相关配置。

bootloader --location=mbr --driveorder=sda --append="crashkernel=auto rhgb quiet"

--location=,指定引导记录被写入的位置.有效的值如下:mbr(缺省),partition(在包含内核的分区的第一个扇区安装引导装载程序)或none(不安装引导装载程序)。

--driveorder,指定在BIOS引导顺序中居首的驱动器。

--append=,指定内核参数.要指定多个参数,使用空格分隔它们。

network     为通过网络的kickstart安装以及所安装的系统配置联网信息。

network --bootproto=dhcp --device=eth0 --onboot=yes --noipv6 --hostname=CentOS6

--bootproto=[dhcp/bootp/static]中的一种，缺省值是dhcp。bootp和dhcp被认为是相同的。

static方法要求在kickstart文件里输入所有的网络信息。

network --bootproto=static --ip=10.0.10.100 --netmask=255.255.255.0 --gateway=10.0.0.2 --nameserver=10.0.0.2

请注意所有配置信息都必须在一行上指定,不能使用反斜线来换行。

--ip=,要安装的机器的IP地址.

--gateway=,IP地址格式的默认网关.

--netmask=,安装的系统的子网掩码.

--hostname=,安装的系统的主机名.

--onboot=,是否在引导时启用该设备.

--noipv6=,禁用此设备的IPv6.

--nameserver=,配置dns解析.

timezone     设置系统时区。timezone --utc Asia/Shanghai

authconfig     系统认证信息。authconfig --enableshadow --passalgo=sha512

设置密码加密方式为sha512 启用shadow文件。

rootpw     root密码

clearpart     清空分区。clearpart --all --initlabel

--all 从系统中清除所有分区，--initlable 初始化磁盘标签

part     磁盘分区。

part /boot --fstype=ext4 --asprimary --size=200

part swap --size=1024

part / --fstype=ext4 --grow --asprimary --size=200

--fstype=,为分区设置文件系统类型.有效的类型为ext2,ext3,swap和vfat。

--asprimary,强迫把分区分配为主分区,否则提示分区失败。

--size=,以MB为单位的分区最小值.在此处指定一个整数值,如500.不要在数字后面加MB。

--grow,告诉分区使用所有可用空间(若有),或使用设置的最大值。

firstboot     负责协助配置redhat一些重要的信息。

firstboot --disable

selinux     关闭selinux。selinux --disabled

firewall     关闭防火墙。firewall --disabled

logging     设置日志级别。logging --level=info

reboot     设定安装完成后重启,此选项必须存在，不然kickstart显示一条消息，并等待用户按任意键后才重新引导，也可以选择halt关机。


```

```
 采用kickstart自动应答程序来实现系统的自动化安装(即静默安装)

 方法：使用kickstart程序或vim来创建ks.cfg自动应答文件。用文件共享服务来共享ks.cfg自动应答文件。

 技巧：linux系统在安装时会自动生成一个anaconda-ks.cfg配置文件，文件位于/root目录中，anaconda-ks.cfg里面记录的就是用户在安装系统时所做的操作（选择语言环境、硬盘分区、安装的软件包、网卡IP设置、主机名、root用户密码、新建普通用户等）。



 具体实施：

   1.修改/root/anaconda-ks.cfg权限为644，复制/root/anaconda-ks.cfg到/var/ftp/ks目录中。

    cd    /root

    chmod  -v  644  anaconda-ks.cfg

    mkdir  -v  /var/ftp/ks

    cp  -v  anaconda-ks.cfg  /var/ftp/ks/ks.cfg



   2.修改/var/lib/tftpboot/pxelinux.cfg/default启动菜单文件

    vim  /var/lib/tftpboot/pxelinux.cfg/default       :set nu 显示行号 

    修改63行内容为append  initrd=initrd.img  method=ftp://192.168.11.11/dvd  ks=ftp://192.168.11.11/ks/ks.cfg

                 说明：ks=是指定ks.cfg自动安装应答文件的功能选项

   3.修改/var/ftp/ks/ks.cfg文件内容

    cat  /var/ftp/ks/ks.cfg       查看ks.cfg自动应答文件

    注释cdrom

    改ip=192.168.11.12

    改hostname=han



   4.采用kickstart软件来生成ks.cfg自动应答文件。



    yum  search  kickstart    搜索kickstart关键字的软件

    yum  install  -y  system-config-kickstart   安装kickstart软件包

    system-config-kickstart     启动kickstart配置程序



     注意：yum源的repo文件中的“仓库标识”必须用[development]才能出现软件包信息

       修改yum源配置文件：

       vim   /etc/yum.repos.d/local.repo   文件内容如下

       [development]

       name=centos 7.5  linux

       baseurl=file:///dvd

       enabled=1

       gpgcheck=0





Linux下的/dev/sr0、/dev/cdrom

/dev/sr0 光驱的设备名

/dev/cdrom 代表光驱



cdrom是sr0的软链接.

<root@localhost dev># ll cdrom

lrwxrwxrwx. 1 root root 3 12月  7 23:15 cdrom -> sr0



sudo mount -o loop a.img /mnt/floppy 系统总是提示我“you must specify the filesystem type” 

blkid

mount -t type -o loop *.img



vim /etc/fstab

0 0 表示的含义是：代表不要做dump备份、不要检验

loop：用来把一个文件当成硬盘分区挂接上系统

/dev/cdrom                                /var/www/html/pub                    iso9660 defaults        0 0



-a 加载文件/etc/fstab中设置的所有设备。 

 mount -a


```

> . loop 设备介绍
> 
> ​ loop 设备是一种伪设备 (pseudo-device)，或者也可以说是仿真设备。它能使我们像块设备一样访问一个文件。
> 
> 在使用之前，一个 loop 设备必须要和一个文件进行连接。这种结合方式给用户提供了一个替代块特殊文件的接口。因此，如果这个文件包含有一个完整的文件系统
> 
> ，那么这个文件就可以像一个磁盘设备一样被 mount 起来。
> 
> 上面说的文件格式，我们经常见到的是 CD 或 DVD 的 ISO 光盘镜像文件或者是软盘 (硬盘) 的 *.img 镜像文件。通过这种 loop mount (回环 mount)的方式，这些镜像文件就可以被 mount 到当前文件系统的一个目录下。
> 
> 至此，顺便可以再理解一下 loop 之含义：对于第一层文件系统，它直接安装在我们计算机的物理设备之上；而对于这种被 mount 起来的镜像文件 (它也包含有文件系统)，它是建立在第一层文件系统之上，这样看来，它就像是在第一层文件系统之上再绕了一圈的文件系统，所以称为 loop。

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211207235209370.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211207235209370.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211207235209370.png)

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211209050215490.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211209050215490.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211209050215490.png)

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211208003456770.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211208003456770.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211208003456770.png)

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211208003745577.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211208003745577.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211208003745577.png)

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211208004401930.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211208004401930.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211208004401930.png)

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211208004629955.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211208004629955.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211208004629955.png)

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211209045343992.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211209045343992.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211209045343992.png)

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211209045534407.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211209045534407.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211209045534407.png)

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211209045827143.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211209045827143.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211209045827143.png)

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211208005445866.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211208005445866.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211208005445866.png)

四、cobbler
=========

官网：[https://cobbler.readthedocs.io/en/latest/quickstart-guide.html#server-and-next-server](https://cobbler.readthedocs.io/en/latest/quickstart-guide.html#server-and-next-server)

[http://cobbler.github.io/](http://cobbler.github.io/)

1. 简介[#](#1-简介)
---------------

Cobbler 是一个 Linux 服务器安装的服务，可以通过网络启动 (PXE) 的方式来快速安装、重装物理服务器和虚拟机，同时还可以管理 DHCP，DNS 等。

Cobbler 可以使用命令行方式管理，也提供了基于 Web 的界面管理工具 (cobbler-web)，还提供了 API 接口，可以方便二次开发使用。

Cobbler 是较早前的 kickstart 的升级版，优点是比较容易配置，还自带 web 界面比较易于管理。

2. 工作原理[#](#2-工作原理)
-------------------

[](https://gitee.com/wuheeee/images/raw/master/images/20190413134111385.png)[![](https://gitee.com/wuheeee/images/raw/master/images/20190413134111385.png)](https://gitee.com/wuheeee/images/raw/master/images/20190413134111385.png)

```
"Server端：

第一步，启动Cobbler服务

第二步，进行Cobbler错误检查，执行cobbler check命令

第三步，进行配置同步，执行cobbler sync命令

第四步，复制相关启动文件文件到TFTP目录中

第五步，启动DHCP服务，提供地址分配

第六步，DHCP服务分配IP地址

第七步，TFTP传输启动文件

第八步，Server端接收安装信息

第九步，Server端发送ISO镜像与Kickstart文件

————————————————

"Client端：

第一步，客户端以PXE模式启动

第二步，客户端获取IP地址

第三步，通过TFTP服务器获取启动文件

第四步，进入Cobbler安装选择界面

第五步，客户端确定加载信息

第六步，根据配置信息准备安装系统

第七步，加载Kickstart文件

第八步，传输系统安装的其它文件

第九步，进行安装系统


```

3. 环境[#](#3-环境)
---------------

```
cat /etc/redhat-release

uname -r



#关闭防火墙

systemctl disable --now firewalld

#关闭selinux

setenforce 0

getenforce



虚拟机网卡采用nat模式(关闭dhcp)



<root@localhost tftpboot># ifconfig ens33|awk -F "[ :]+" 'NR==2 {print $3}'

192.168.19.100




```

4. 安装 cobbler 以及相关的软件[#](#4-安装cobbler以及相关的软件)
---------------------------------------------

参考：[https://www.cnblogs.com/linuxliu/p/7668048.html](https://www.cnblogs.com/linuxliu/p/7668048.html)

参考：[https://blog.csdn.net/yuanfangPOET/article/details/89279733](https://blog.csdn.net/yuanfangPOET/article/details/89279733)

参考：[https://www.cnblogs.com/wjlovezzd/p/13178561.html](https://www.cnblogs.com/wjlovezzd/p/13178561.html)

参考：[https://blog.csdn.net/weixin_41224474/article/details/106319149](https://blog.csdn.net/weixin_41224474/article/details/106319149)

参考：[https://blog.51cto.com/u_5001660/2530428](https://blog.51cto.com/u_5001660/2530428)

### 1. 命令管理[#](#1-命令管理)

```
cobbler <distro|profile|system|repo|p_w_picpath|mgmtclass|package|file> ...

        [add|edit|copy|getks*|list|remove|rename|report] [options|--help]

cobbler <aclsetup|buildiso|import|list|replicate|report|reposync|sync|validateks|version|signature|get-loaders|hardlink> [options|--help]



cobbler check    核对当前设置是否有问题

cobbler list     列出所有的cobbler元素cobbler

report           列出元素的详细信息

cobbler sync     同步配置到数据目录,更改配置最好都要执行下

cobbler reposync    同步yum仓库

cobbler distro   查看导入的发行版系统信息

cobbler system   查看添加的系统信息cobbler

profile  查看配置信息




```

### 2. 安装相关依赖包[#](#2-安装相关依赖包)

```
cobbler          #cobbler程序包

cobbler-web   #cobbler的web服务包

pykickstart     #cobbler检查kickstart语法错误



# yum -y install epel-release.noarch

# yum install  -y cobbler cobbler-web pykickstart httpd dhcp tftp-server debmirror xinetd



查看cobbler安装的部分文件



# rpm -ql cobbler  # 查看安装的文件，下面列出部分。

/etc/cobbler                  # 配置文件目录

/etc/cobbler/settings         # cobbler主配置文件，这个文件是yaml格式，cobbler是python写的程序。

/etc/cobbler/dhcp.template    # dhcp服务的配置模板

/etc/cobbler/tftpd.template   # tftp服务的配置模板

/etc/cobbler/rsync.template   # rsync服务的配置模板

/etc/cobbler/iso              # iso模板配置文件目录

/etc/cobbler/pxe              # pxe模板文件目录

/etc/cobbler/power            # 电源的配置文件目录

/etc/cobbler/users.conf       # web服务授权配置文件

/etc/cobbler/users.digest     # web访问的用户名密码配置文件

/etc/cobbler/dnsmasq.template # DNS服务的配置模板

/etc/cobbler/modules.conf     # cobbler模块配置文件

/var/lib/cobbler              # cobbler数据目录

/var/lib/cobbler/config       # 配置文件

/var/lib/cobbler/kickstarts   # 默认存放kickstart文件

/var/lib/cobbler/loaders      # 存放的各种引导程序

/var/www/cobbler              # 系统安装镜像目录

/var/www/cobbler/ks_mirror    # 导入的系统镜像列表

/var/www/cobbler/p_w_picpaths       # 导入的系统镜像启动文件

/var/www/cobbler/repo_mirror  # yum源存储目录

/var/log/cobbler              # 日志目录

/var/log/cobbler/install.log  # 客户端系统安装日志

/var/log/cobbler/cobbler.log  # cobbler日志



启动服务

# systemctl restart httpd

# systemctl enable cobblerd httpd --now

# netstat -naltp |grep 25151




```

### 3. 检测 cobbler[#](#3-检测cobbler)

```
[root@n1 ~]# cobbler check

 

The following are potential configuration items that you may want to fix:

 

1 : The 'server' field in /etc/cobbler/settings must be set to something other than localhost, or kickstarting features will not work.  This should be a resolvable hostname or IP for the boot server as reachable by all machines that will use it.

 

2 : For PXE to be functional, the 'next_server' field in /etc/cobbler/settings must be set to something other than 127.0.0.1, and should match the IP of the boot server on the PXE network.

 

3 : change 'disable' to 'no' in /etc/xinetd.d/tftp

 

4 : Some network boot-loaders are missing from /var/lib/cobbler/loaders, you may run 'cobbler get-loaders' to download them, or, if you only want to handle x86/x86_64 netbooting, you may ensure that you have installed a *recent* version of the syslinux package installed and can ignore this message entirely.  Files in this directory, should you want to support all architectures, should include pxelinux.0, menu.c32, elilo.efi, and yaboot. The 'cobbler get-loaders' command is the easiest way to resolve these requirements.

 

5 : enable and start rsyncd.service with systemctl

 

6 : debmirror package is not installed, it will be required to manage debian deployments and repositories

 

7 : The default password used by the sample templates for newly installed machines (default_password_crypted in /etc/cobbler/settings) is still set to 'cobbler' and should be changed, try: "openssl passwd -1 -salt 'random-phrase-here' 'your-password-here'" to generate new one

 

8 : fencing tools were not found, and are required to use the (optional) power management features. install cman or fence-agents to use them

 

Restart cobblerd and then run 'cobbler sync' to apply changes.


```

### 4. 按照指示修改[#](#4-按照指示修改)

```
1.修改/etc/cobbler/settings中的"server"字段为提供cobbler服务的主机的IP或主机名

[root@n1 ~]# sed -i 's/^server: 127.0.0.1/server: 192.168.19.100/' /etc/cobbler/settings

[root@n1 ~]# grep "^server" /etc/cobbler/settings

server: 192.168.231.60

 

2.修改/etc/cobbler/settings中"next_server"为PXE网络上 TFTP Server 的IP地址（这里PTFTP和cobbler在同一主机）

[root@n1 ~]# sed -i 's/^next_server: 127.0.0.1/next_server: 192.168.19.100/' /etc/cobbler/settings

[root@n1 ~]# grep "^next_server" /etc/cobbler/settings

next_server: 192.168.231.60

 

修改一些其他的配置



        vim /etc/cobbler/settings



        manage_dhcp: 1  #用cobbler管理dhcp（如果是0，就是）

        manage_tftpd: 1

        pxe_just_once: 1   #防止循环安装系统，适用于服务器第一启动选项是pxe启动 

 

3.修改/etc/xinet.d/tftp文件中的disable的参数为no

[root@n1 ~]# cp /etc/xinetd.d/tftp{,.bak}  #备份

[root@n1 ~]# vim /etc/xinetd.d/tftp

        disable                 = no

 

4. 向代码主站发起必备数据,执行命令(可能报错)

[root@n1 ~]# cobbler get-loaders

 

如果报错，执行下面的操作

[root@n1 ~]# yum -y install syslinux

[root@n1 ~]# cp/usr/share/syslinux/pxelinux.0 /var/lib/cobbler/loaders/

[root@n1 ~]# cp/usr/share/syslinux/menu.c32 /var/lib/cobbler/loaders/

[root@n1 ~]# systemctl restart cobblerd

[root@n1 ~]# cobbler get-loaders    #再次执行则成功

关闭非常规系统以及32为系统安装

[root@n1 ~]#  vim /etc/debmirror.conf

#@dists="sid";

#@arches="i386";





5.启动rsync服务（cobber 借助rsync目录之间的同步）

[root@n1 ~]# systemctl start rsyncd.service

[root@n1 ~]# systemctl enable rsyncd.service

 

6.安装包用来管理部署和存储库，这里不需要，就不安装了

 

7.修改cobbler中的加密密码（即为自动安装系统后的root登录密码）

#生成加密密码openssl passwd -1 -salt 'random-phrase-here' 'your-password-here'

[root@n1 ~]# openssl passwd -1 -salt '123456' 'realxw'

$1$123456$jMyuS0sKNP4A/OOlui3lR/

[root@n1 ~]# vim /etc/cobbler/settings

default_password_crypted: "$1$123456$jMyuS0sKNP4A/OOlui3lR/"



[root@n1 ~]# cobbler check

[root@n1 ~]# cobbler sync

[root@n1 ~]# systemctl restart cobblerd

[root@n1 ~]# systemctl status cobblerd





8.安装cman fence-agents（隔离机制，为防止重复安装）

[root@n1 ~]# yum install cman fence-agents -y








```

报错：

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211210160438597.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211210160438597.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211210160438597.png)

```
yum -y install syslinux

cp/usr/share/syslinux/pxelinux.0 /var/lib/cobbler/loaders/

cp/usr/share/syslinux/menu.c32 /var/lib/cobbler/loaders/

systemctl restart cobblerd

cobbler get-loaders    #再次执行则成功


```

### 4. 配置 dhcp[#](#4-配置dhcp)

```
[root@n1 ~]# cp /etc/cobbler/dhcp.template{,.bak}

[root@n1 ~]# vim /etc/cobbler/dhcp.template

subnet 192.168.19.0 netmask 255.255.255.0 {

     option routers             192.168.19.2;

     option domain-name-servers 223.5.5.5;

     option subnet-mask         255.255.255.0;

     range dynamic-bootp        192.168.19.20 192.168.19.50;

     default-lease-time         21600;

     max-lease-time             43200;

     next-server                $next_server;



    

重启服务并同步配置，改完dhcp必须要sync同步配置

[root@n1 ~]# cobbler sync

[root@n1 ~]# cobbler check

[root@n1 ~]# systemctl restart cobblerd.service 

[root@n1 ~]# netstat -lnup|grep dhcp


```

### 5. 重启所有服务，防止一些服务没有开启[#](#5-重启所有服务防止一些服务没有开启)

```
systemctl enable dhcpd

systemctl enable rsyncd.service

systemctl enable tftp.service

systemctl enable httpd.service

systemctl enable cobblerd.service



systemctl restart tftp.service

systemctl restart dhcpd.service

systemctl restart rsyncd.service

systemctl restart httpd.service

systemctl restart cobblerd.service




```

### 6. 挂载光盘镜像并导入[#](#6-挂载光盘镜像并导入)

```
<root@localhost ~># lsblk

NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT

sda               8:0    0   15G  0 disk

├─sda1            8:1    0    1G  0 part /boot

└─sda2            8:2    0   14G  0 part

  ├─centos-root 253:0    0 12.5G  0 lvm  /

  └─centos-swap 253:1    0  1.5G  0 lvm  [SWAP]

sr0              11:0    1  4.4G  0 rom

sr1              11:1    1 10.1G  0 rom

sr2              11:2    1  3.7G  0 rom

<root@localhost ~># mount /dev/sr1 /var/www/html/CentOS8

mount: /dev/sr1 is write-protected, mounting read-only

<root@localhost ~># mount /dev/sr0 /var/www/html/CentOS7

mount: /dev/sr0 is write-protected, mounting read-only

<root@localhost ~># mount /dev/sr2 /var/www/html/CentOS6

mount: /dev/sr2 is write-protected, mounting read-only



<root@localhost ~># lsblk



<root@localhost ~># cobbler import --path=/var/www/html/CentOS7 --name=Centos-7.9 --arch=x86_64

<root@localhost ~># cobbler import --path=/var/www/html/CentOS6 --name=Centos-6.9 --arch=x86_64

<root@localhost ~># cobbler import --path=/var/www/html/CentOS8 --name=Centos-8.5 --arch=x86_64



[root@n1 ~]# mount -t iso9660 -o loop /soft/CentOS-7-x86_64-Minimal-1804.iso /iso/   #这里是CentOS7.5mini镜像





# --path 镜像路径

# --name 为安装源定义一个名字

# --arch 指定安装源是32位、64位、ia64, 目前支持的选项有: x86│x86_64│ia64

# 安装源的唯一标示就是根据name参数来定义，本例导入成功后，安装源的唯一标示就是：CentOS-7.1-x86_64，如果重复，系统会提示导入失败

# 导入镜像的位置

[root@n1 ~]# cobbler import --path=/mnt --name=Centos-7.2 --arch=x86_64 # cobbler导入镜像







<root@localhost www># cobbler import --path=/var/www/html/CentOS7 --name=Centos-7.9 --arch=x86_64

task started: 2021-12-10_171820_import

task started (id=Media import, time=Fri Dec 10 17:18:20 2021)

Found a candidate signature: breed=redhat, version=rhel6

Found a candidate signature: breed=redhat, version=rhel7

Found a matching signature: breed=redhat, version=rhel7

Adding distros from path /var/www/cobbler/ks_mirror/Centos-7.9-x86_64:

creating new distro: Centos-7.9-x86_64

trying symlink: /var/www/cobbler/ks_mirror/Centos-7.9-x86_64 -> /var/www/cobbler/links/Centos-7.9-x86_64

creating new profile: Centos-7.9-x86_64

associating repos

checking for rsync repo(s)

checking for rhn repo(s)

checking for yum repo(s)

starting descent into /var/www/cobbler/ks_mirror/Centos-7.9-x86_64 for Centos-7.9-x86_64

processing repo at : /var/www/cobbler/ks_mirror/Centos-7.9-x86_64

need to process repo/comps: /var/www/cobbler/ks_mirror/Centos-7.9-x86_64

looking for /var/www/cobbler/ks_mirror/Centos-7.9-x86_64/repodata/*comps*.xml

Keeping repodata as-is :/var/www/cobbler/ks_mirror/Centos-7.9-x86_64/repodata




```

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211210163543886.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211210163543886.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211210163543886.png)

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211210172253984.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211210172253984.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211210172253984.png)

### 7. 导入完镜像以后，那么就使查看下 cobbler[#](#7-导入完镜像以后那么就使查看下cobbler)

```
cobbler commands介绍

cobbler check 核对当前设置是否有问题

cobbler list 列出所有的cobbler元素

cobbler report 列出元素的详细信息

cobbler sync 同步配置到数据目录,更改配置最好都要执行下

cobbler reposync 同步yum仓库

cobbler distro 查看导入的发行版系统信息

cobbler system 查看添加的系统信息

cobbler profile 查看配置信息


```

```
<root@localhost tftpboot># tree /var/lib/tftpboot/

/var/lib/tftpboot/

├── boot

│   └── grub

│       └── menu.lst

├── etc

├── grub

│   ├── efidefault

│   ├── grub-x86_64.efi

│   ├── grub-x86.efi

│   └── images -> ../images

├── images

│   └── Centos-7.9-x86_64

│       ├── initrd.img

│       └── vmlinuz

├── images2

├── ks1.cfg

├── ks2.cfg

├── ks3.cfg

├── ks.cfg

├── ldlinux

├── libcom32

├── libutil

├── memdisk

├── menu.c32

├── ppc

├── pxelinux.0

├── pxelinux.cfg

│   └── default

├── s390x

│   └── profile_list

├── vesamenu.c32

└── yaboot





<root@localhost www># pwd

/var/www

<root@localhost www># cobbler list

distros:

   Centos-7.9-x86_64



profiles:

   Centos-7.9-x86_64



systems:



repos:



images:



mgmtclasses:



packages:



files:



<root@localhost www># cobbler distro list

   Centos-7.9-x86_64

<root@localhost www># cobbler profile list

   Centos-7.9-x86_64

# 查看导入信息及默认ks文件

# 第一次导入系统镜像后，cobbler会给镜像指定一个默认的kickstart自动安装文件(/var/lib/cobbler/kickstarts/sample_end.ks)

<root@localhost tftpboot># cobbler report

<root@localhost tftpboot># cobbler report

distros:

==========

Name                           : Centos-7.9-x86_64

Architecture                   : x86_64

TFTP Boot Files                : {}

Breed                          : redhat

Comment                        :

Fetchable Files                : {}

Initrd                         : /var/www/cobbler/ks_mirror/Centos-7.9-x86_64/images/pxeboot/initrd.img

Kernel                         : /var/www/cobbler/ks_mirror/Centos-7.9-x86_64/images/pxeboot/vmlinuz

Kernel Options                 : {}

Kernel Options (Post Install)  : {}

Kickstart Metadata             : {'tree': 'http://@@http_server@@/cblr/links/Centos-7.9-x86_64'}

Management Classes             : []

OS Version                     : rhel7

Owners                         : ['admin']

Red Hat Management Key         : <<inherit>>

Red Hat Management Server      : <<inherit>>

Template Files                 : {}





profiles:

==========

Name                           : Centos-7.9-x86_64

TFTP Boot Files                : {}

Comment                        :

DHCP Tag                       : default

Distribution                   : Centos-7.9-x86_64

Enable gPXE?                   : 0

Enable PXE Menu?               : 1

Fetchable Files                : {}

Kernel Options                 : {}

Kernel Options (Post Install)  : {}

# # 默认ks文件，这里需要修改为我们自己配置好的ks文件（或者删除重新写）

Kickstart                      : /var/lib/cobbler/kickstarts/sample_end.ks

Kickstart Metadata             : {}

Management Classes             : []

Management Parameters          : <<inherit>>

Name Servers                   : []

Name Servers Search Path       : []

Owners                         : ['admin']

Parent Profile                 :

Internal proxy                 :

Red Hat Management Key         : <<inherit>>

Red Hat Management Server      : <<inherit>>

Repos                          : []

Server Override                : <<inherit>>

Template Files                 : {}

Virt Auto Boot                 : 1

Virt Bridge                    : xenbr0

Virt CPUs                      : 1

Virt Disk Driver Type          : raw

Virt File Size(GB)             : 5

Virt Path                      :

Virt RAM (MB)                  : 512

Virt Type                      : kvm





systems:

==========



repos:

==========



images:

==========



mgmtclasses:

==========



packages:

==========



files:

==========




```

### 8. 导入 kickstarts 配置文件[#](#8-导入kickstarts配置文件)

```
# 查看ks文件位置

<root@localhost tftpboot># ls /var/lib/cobbler/kickstarts

default.ks    esxi5-ks.cfg      legacy.ks     sample_autoyast.xml  sample_esx4.ks   sample_esxi5.ks  sample.ks        sample.seed

esxi4-ks.cfg  install_profiles  pxerescue.ks  sample_end.ks        sample_esxi4.ks  sample_esxi6.ks  sample_old.seed  sample.seed.28



<root@localhost tftpboot># cobbler profile list

   Centos-7.9-x86_64

   

"方法一："   

# 删除应答文件   

<root@localhost tftpboot># cobbler profile remove --name Centos-7.9-x86_64

<root@localhost tftpboot># cobbler profile list

<root@localhost tftpboot># cobbler report

<root@localhost kickstarts># cobbler profile add --name=centos7.6-basic --distro=Centos-7.9-x86_64 --kickstart=/var/lib/cobbler/kickstarts/centos7.ks

<root@localhost kickstarts># cobbler profile list

   centos7.6-basic



"方法二："   



# 配置自定义ks文件，以sample_end.ks为模板

[root@n1 kickstarts]# cp sample_end.ks CentOS7.5-x86_64.ks

[root@n1 kickstarts]# vim CentOS7.5-x86_64.ks    #只修改如下内容

firewall --disabled     #关闭防火墙

timezone  Asia/Shanghai  #设置时区

# autopart    #把自动分区注释,手动如下设置

part /boot --fstype=ext4 --asprimary --size=200

part swap --asprimary --size=1024

part / --fstype=ext4 --grow --asprimary --size=10240

%packages   #安装需要安装的软件包

$SNIPPET('func_install_if_enabled')  #这里默认

%end

 

解释

--asprimary,强迫把分区分配为主分区,否则提示分区失败.

--fstype=,为分区设置文件系统类型.有效的类型为ext2,ext3,swap和vfat

--grow  让分区自动增长利用可用的磁盘空间，或是增长到设置的maxsize值；

--size=  设置分区的最小值，默认单位为M，但是不能写单位；

 

 %packages部分,这部分选择需要安装的软件包.







如想详细了解怎样配置kickstart，可参考  https://blog.csdn.net/taiyang1987912/article/details/42176709



编辑profile，修改关联的ks文件（指定自定义ks文件）



[root@n1 ~]# cobbler profile edit --name=CentOS7.5-x86_64 --kickstart=/var/lib/cobbler/kickstarts/CentOS7.5-x86_64.ks



检查ks命令



# 写完 ks 文件之后，先通过 validateks 测试一下有没有语法错误

# cobbler validateks

# 通过下面这个命令查看 ks 文件，发现一些逻辑上的问题

# cobbler system getks --name=test



<root@localhost kickstarts># cobbler profile edit --name=Centos-7.9-x86_64 --kickstart=/var/lib/cobbler/kickstarts/CentOS7.9-x86_64.ks

<root@localhost kickstarts># cobbler profile report





同步数据



[root@n1 ~]# cobbler sync



[root@n1 ~]# systemctl restart cobblerd.service


```

### 9. 修改 PXE 默认启动选项[#](#9-修改pxe默认启动选项)

默认情况下 PXE 启动的是 Local（启动后会出现 local 和自定义 CentOS7.9-x86_64 两个选项，需手动，要无人工干涉，就需要修改）

*   修改前

```
[root@n1 ~]# cd /var/lib/tftpboot/pxelinux.cfg/

[root@n1 pxelinux.cfg]# cat default

DEFAULT menu

PROMPT 0

MENU TITLE Cobbler | http://cobbler.github.io/

TIMEOUT 200

TOTALTIMEOUT 6000

ONTIMEOUT local

LABEL local

       MENU LABEL (local)

       MENU DEFAULT

       LOCALBOOT -1

LABEL CentOS7.5-x86_64

       kernel /images/CentOS7.5-x86_64/vmlinuz

       MENU LABEL CentOS7.5-x86_64

       append initrd=/images/CentOS7.5-x86_64/initrd.img ksdevice=bootif lang=  kssendmac text  ks=http://192.168.231.60/cblr/svc/op/ks/profile/CentOS7.5-x86_64

       ipappend 2

MENU end


```

*   修改后

```
[root@n1 pxelinux.cfg]# cat default

DEFAULT menu

PROMPT 0

MENU TITLE Cobbler | http://cobbler.github.io/

TIMEOUT 200                # 自动安装开始时间(#菜单停留超时时间)

TOTALTIMEOUT 6000

ONTIMEOUT CentOS7.5-x86_64       # 改为 CentOS7.9-x86_64



LABEL centos7.6-basic

        kernel /images/Centos-7.9-x86_64/vmlinuz

        MENU LABEL centos7.6-basic

        MENU DEFAULT

        append initrd=/images/Centos-7.9-x86_64/initrd.img ksdevice=bootif lang=  kssendmac text  ks=http://192.168.19.100/cblr/svc/op/ks/profile/centos7.6-basic

        ipappend 2



MENU end


```

要设置足够的内存，否则会出现如下错误：

[](https://gitee.com/wuheeee/images/raw/master/images/f5552221a47432e40334eb6f9e9d62cc.png)[![](https://gitee.com/wuheeee/images/raw/master/images/f5552221a47432e40334eb6f9e9d62cc.png)](https://gitee.com/wuheeee/images/raw/master/images/f5552221a47432e40334eb6f9e9d62cc.png)

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211210183329557.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211210183329557.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211210183329557.png)

上图中网址也可以定制为我们自己的

```
说明：在client端系统安装时，可以在cobbler服务端上查看日志/var/log/messages，观察安装的每一个流程



[root@localhost cobbler]# tail -f /var/log/messages   查看日志

[root@localhost cobbler]# vim /etc/cobbler/pxe/pxedefault.template

MENU TITLE Cobbler | I'm here # 修改这里为你想修改的内容

[root@localhost cobbler]# cobbler sync # 同步之后就可以看到效果了


```

**以上步骤均不需手动操作，自动安装（备注：如果被安装的设备有两个网卡，需要手动选择正确的网卡。如果有多块硬盘，需要手动选择安装的盘，无亲自测试，建议安装对的时候安装一块盘，待系统安装完成后，用脚本进行挂载和格式化。）。**

[](https://gitee.com/wuheeee/images/raw/master/images/image-20211210190657028.png)[![](https://gitee.com/wuheeee/images/raw/master/images/image-20211210190657028.png)](https://gitee.com/wuheeee/images/raw/master/images/image-20211210190657028.png)

### 10. centos 应答文件参考[#](#10-centos-应答文件参考)

```
<root@localhost html># cat ks-centos6.cfg

#platform=x86, AMD64, 或 Intel EM64T

#version=DEVEL

# Install OS instead of upgrade

install

# Keyboard layouts

keyboard 'us'

# Root password (111111)

rootpw --iscrypted $1$DaMGwMzL$.v/4oq8xO2g.rxXDVjft10

# System language

lang en_US

# System authorization information

auth  --useshadow  --passalgo=sha512

# Use graphical install

graphical

firstboot --disable

# SELinux configuration

selinux --disabled



# Firewall configuration

firewall --disabled

# Reboot after installation

reboot

# System timezone

timezone Asia/Shanghai

# Use network installation

# url --url="http://192.168.19.100/CentOS6"

url --url="http://192.168.19.100/cobbler/ks_mirror/Centos-6.9-x86_64/"

# System bootloader configuration

bootloader --location=mbr

# Clear the Master Boot Record

zerombr

# Partition clearing information

clearpart --all --initlabel

# Disk partitioning information

part /boot --fstype="ext4" --size=1024

part swap --fstype="swap" --size=4096

part / --fstype="ext4" --grow --size=1



%post

yum -y install httpd

echo 1111 > root.txt

%end





%packages

@core

@server-policy

@workstation-policy

%end





<root@localhost html># cat ks-centos7.cfg

#platform=x86, AMD64, 或 Intel EM64T

#version=DEVEL

# Install OS instead of upgrade

install

# Keyboard layouts

keyboard 'us'

# Root password

rootpw --iscrypted $1$DaMGwMzL$.v/4oq8xO2g.rxXDVjft10

# System language

lang en_US

# System authorization information

auth  --useshadow  --passalgo=sha512

# Use graphical install

graphical

firstboot --disable

# SELinux configuration

selinux --disabled





# Firewall configuration

firewall --disabled

# Reboot after installation

reboot

# System timezone

timezone Asia/Shanghai

# Use network installation

# url --url="http://192.168.19.100/CentOS7"

url --url="http://192.168.19.100/cobbler/ks_mirror/Centos-7.9-x86_64/"

# System bootloader configuration

bootloader --location=mbr

# Clear the Master Boot Record

zerombr

# Partition clearing information

clearpart --all --initlabel

autopart --type=lvm

# Disk partitioning information

#part /boot --fstype="xfs" --size=1024

#part swap --fstype="swap" --size=4096

#part / --fstype="xfs" --grow --size=1



%post

yum -y install httpd

echo 1111 > /root/root.txt

%end



%packages

@^minimal

@core

@development

chrony

kexec-tools



%end







<root@localhost html># cat ks-centos8.cfg

#version=RHEL8

# Reboot after installation

reboot

# Use graphical install

graphical



repo -- --baseurl=http://192.168.19.100/cobbler/ks_mirror/Centos-8.5-x86_64/AppStream



%packages

@^minimal-environment

@legacy-unix

@standard

@system-tools

kexec-tools



%end



# Keyboard layouts

keyboard --vckeymap=us --xlayouts='us'

# System language

lang en_US.UTF-8

# Root password (111111)

rootpw --iscrypted $1$DaMGwMzL$.v/4oq8xO2g.rxXDVjft10

# Firewall configuration

firewall --disabled

# Network information

network  --hostname=localhost.localdomain



# Use network installation

# url --url="http://192.168.19.100/CentOS8"

url --url="http://192.168.19.100/cobbler/ks_mirror/Centos-8.5-x86_64/"

# System authorization information

auth --useshadow --passalgo=sha512

# SELinux configuration

selinux --disabled



firstboot --disable



ignoredisk --only-use=nvme0n1

# System bootloader configuration

bootloader --append="crashkernel=auto" --location=mbr --boot-drive=nvme0n1

# Clear the Master Boot Record

zerombr

# Partition clearing information

clearpart --all --initlabel

autopart --type=lvm

# Disk partitioning information

#part /date --fstype="xfs" --size=8888

#part /boot --fstype="xfs" --size=1024

#part swap --fstype="swap" --size=4096

#part / --fstype="xfs" --size=6470



# System timezone

timezone Asia/Shanghai



# Root password

rootpw --iscrypted $1$DaMGwMzL$.v/4oq8xO2g.rxXDVjft10



%addon com_redhat_kdump --enable --reserve-mb='auto'



%end


```

### 11. 通过 MAC 地址定制化安装[#](#11-通过mac地址定制化安装)

[](https://gitee.com/wuheeee/images/raw/master/images/795249-20171015161724996-958508827.png)[![](https://gitee.com/wuheeee/images/raw/master/images/795249-20171015161724996-958508827.png)](https://gitee.com/wuheeee/images/raw/master/images/795249-20171015161724996-958508827.png)

配置定制化安装（需要验证，后续验证后添加验证结果）

```
[root@localhost cobbler]# cobbler system add \   

--name=linux-web01 \

--mac=00:0C:29:3B:03:9B \

--profile=Centos-7.2-x86_64 \

--ip-address=10.0.0.200 \

--subnet=255.255.255.0 \

--gateway=10.0.0.2 \

--interface=eth0 \

--static=1 \

--hostname=linux-web01 \

--name-servers="10.0.0.2" \

--kickstart=/var/lib/cobbler/kickstarts/Centos7.2-x86_64.cfg





system add  #  添加定制系统



name  # 定制系统名称



mac # mac地址



profile #指定profile



ip-address # 指定IP地址



subnet # 指定子网掩码



gateway # 指定网关



interface # 指定网卡，eth0上面配置已经修改，centos7默认网卡名称不是eth0



static # 1表示启用静态IP



hostname # 定义hostname



name-server # dns服务器



kickstart # 指定ks文件



配置成功后我们可以查看到刚才定制的系统

[root@localhost cobbler]# cobbler system list

linux-web01



接下来我们创建一个虚拟机，mac地址为00:0C:29:3B:03:9B，启动后你就会发现自动进入安装系统了，等安装完以后，所有的配置都和我们当初设置的一样。


```

### 12. 使用 koan 实现重新安装系统[#](#12-使用koan实现重新安装系统)

在客户端安装 koan（要配置好源）

```
[root@localhost ~]# rpm -ivh http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-10.noarch.rpm

[root@localhost ~]# yum install koan


```

查看 cobbler 上的配置文件

```
1 [root@localhost ~]# koan --server=10.0.0.101 --list=profiles

2 - looking for Cobbler at http://10.0.0.101:80/cobbler_api

3 Centos-7.2-x86_64


```

重新安装客户端系统

```
[root@localhost ~]# koan --replace-self --server=10.0.0.101 --profile=webserver1


```

重启系统后会自动重装系统