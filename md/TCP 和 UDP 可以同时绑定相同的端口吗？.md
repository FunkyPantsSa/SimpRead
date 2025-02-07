> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [baijiahao.baidu.com](https://baijiahao.baidu.com/s?id=1756774855186995490&wfr=spider&for=pc)

TCP 和 UDP 可以同时绑定相同的端口吗？解答这个问题之前，我们需要先来了解什么是 TCP 和 UDP，什么又是网络端口。

![](https://pics6.baidu.com/feed/3b87e950352ac65cb96c768813df181a92138a9d.jpeg@f_auto?token=9ea723ac7eda45b68214854083c9e6a7)

### 一、TCP 与 UDP 介绍  

TCP 和 UDP 是 IP（Internet Protocol）的独立的两个协议，他们都工作在 OSI 模型中的网络层。其中 TCP 和 UDP 最大的区别就是面向连接和面向无连接。

**TCP**

当需要传输的数据的可靠性非常重要的时候，我们一般使用 TCP 进行传输，因为 TCP 协议传输的数据是按照顺序依次传输。如果数据接收方未收到发送方传输的数据，TCP 会在特定时间之后重新发包。这就是我们常说的丢包重传机制，还有就是拥塞控制、流量控制等，TCP 的可靠性正是因为有这些特性。

**UDP**

UDP（User Datagram Protocol）是一种面向无连接的服务，UDP 的数据将不像是 TCP 那样保证按序传输，接收方无论收没收到数据都不会重传，因此 UDP 相对于 TCP 有更低的延迟。在时间优先级高于数据可靠性的应用中，UDP 更为常用，例如平时使用的视频通话、网络游戏等。因为对于这些应用来说，时间比数据的一致性更为重要。

### 二、什么是网络端口？端口有什么作用？

我们的电脑上有许多的通信程序，当我们的电脑收到数据包之后，数据包是如何精准的分配至不同的应用的呢？我们可以这样理解，在网络中的 IP 地址相当于我们现实生活中的小区名，端口号就像是具体的门牌号。

![](https://pics7.baidu.com/feed/377adab44aed2e737f9136c56a2c0b8086d6fa9d.png@f_auto?token=e86fe93bc79fc8fdb9c05a22276afcb1)

端口的作用是让应用层的各种应用进程都能将其数据通过端口向下交付给传输层，以及让传输层知道应当将其报文段中的数据向上通过端口交付给应用层的进程。为了对端口进行区分，将每个端口进行了编号，这就是端口号。当我们将数据从一台设备发送到另一台设备时，它会转到特定的 TCP 或 UDP 端口，具体取决于我们用于通信的协议。

### 三、TCP 和 UDP 的 Socket 可以绑定同一个端口吗？

TCP 与 UDP 服务端网络都会调用 bind 绑定端口。

![](https://pics2.baidu.com/feed/c9fcc3cec3fdfc032e84ace038122d9fa5c226a7.jpeg@f_auto?token=bbbf2e42c7b74a8f3467fda62ded3297)TCP 网络编程![](https://pics1.baidu.com/feed/77c6a7efce1b9d165e1af87819f31e848d546487.jpeg@f_auto?token=fb3811b147808c2daf4cec878fa9413b)UDP 网络编程

TCP 和 UDP 端口彼此不相关。TCP 端口由 TCP 堆栈解释，而 UDP 堆栈解释 UDP 端口。端口是多路复用连接的一种方式，以便多个设备可以连接到一个节点。因此，从技术上讲，更高级别的协议可以使用相同或不同的 TCP 和 UDP 端口号。另一方面，一台计算机可以同时使用相同的 TCP 和 UDP 端口号与两个不同的服务进行通信。  

![](https://pics2.baidu.com/feed/29381f30e924b8991732ff416f28b79e0a7bf673.png@f_auto?token=8baf570de4457b390e37efe7a0948bfb)

如上图， TCP/UDP 各自的端口号是相互独立的， TCP 有一个 80 号端口，UDP 也可以拥有一个 80 号端口，两者并不冲突。

所以，TCP 和 UDP 是可以同时绑定相同的端口的。

TCP 和 UDP 传输协议，在内核中是由两个完全独立的软件模块实现的。

当主机收到数据包后，可以在 IP 包头的 “协议号” 字段知道该数据包是 TCP 还是 UDP，所以可以根据这个信息确定送给哪个模块（TCP/UDP）处理，送给 TCP/UDP 模块的报文根据 “端口号” 确定送给哪个应用程序处理。

因此， TCP/UDP 各自的端口号也相互独立，互不影响。

**客户端的端口可以重复使用吗？**

只要客户端连接的不是相同的服务器，内核是允许端口重复使用的。TCP 连接由四元组（源 IP 地址，源端口，目的 IP 地址，目的端口）唯一确认的，四元组其中任何一个元素改变，就表示不同的 TCP 连接。

假如客户使用端口 1 与服务器 A 建立了连接，客户端也可以使用端口 1 与服务器 B 建立连接，即使客户端的端口号相同，但因四元组信息发生变化，并不会导致连接冲突。

**多个 TCP 服务进程可以绑定同一个端口吗？**

若多个 TCP 服务进程同时绑定相同的 IP 地址和端口，那么执行 bind() 时候就会报错 “Address already in use”；若 TCP 服务进程只是绑定相同的端口，但绑定的 IP 地址不同，那么则不会报错。

举报 / 反馈