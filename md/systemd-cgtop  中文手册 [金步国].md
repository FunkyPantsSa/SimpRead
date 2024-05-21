> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jinbuguo.com](https://www.jinbuguo.com/systemd/systemd-cgtop.html)

> systemd-cgtop 中文手册 [金步国作品集]

名称
--

systemd-cgtop — 按照资源使用率从高到低的顺序显示控制组

大纲
--

`systemd-cgtop` [OPTIONS...] [GROUP]

描述 [¶](#%E6%8F%8F%E8%BF%B0 "Permalink to this headline")
--------------------------------------------------------

**systemd-cgtop** 按照资源使用率 (CPU, 内存, 磁盘吞吐率) 从高到低的顺序显示系统中的控制组(Control Group)。 显示的列表每个一段时间进行一次刷新(默认一秒刷新一次)， 输出的列表风格类似于 [top(1)](http://man7.org/linux/man-pages/man1/top.1.html) 。 如果明确指定了 [GROUP] 参数 (控制组路径)， 那么将仅显示指定控制组路径下的单元。

如果 **systemd-cgtop** 并未连接到某个终端 (TTY)， 那么将不会显示列标题，并且在输出列表之后立即退出 (也就是不会重复刷新)。 不过如果明确设置了 `--iterations=` 选项，那么还是会在刷新指定的次数之后再退出。 此模式经常用于脚本环境。

资源占用仅根据控制组的对应的层次进行统计。 具体就是， CPU 占用仅根据控制组的 "`cpuacct`" 层次进行统计。 内存占用仅根据控制组的 "`memory`" 层次进行统计。 磁盘 I/O 占用仅根据控制组的 "`blkio`" 层次进行统计。 如果需要针对特定的单元监视这些资源的占用状况，可以将 `CPUAccounting=1`, `MemoryAccounting=1`, `BlockIOAccounting=1` 添加到特定的单元文件中。详见 [systemd.resource-control(5)](https://www.jinbuguo.com/systemd/systemd.resource-control.html#) 手册。

CPU 占用值介于 0 与 100*CPU 总核数 之间。 例如，在一个拥有 8 颗 CPU 核心的系统上， CPU 占用值将会介于 0% 与 800% 之间。 CPU 总核数可以从 "`/proc/cpuinfo`" 中获取。

再次强调，除非在单元文件中设置了 "`CPUAccounting=1`","`MemoryAccounting=1`","`BlockIOAccounting=1`" 指令， 否则不会针对单个服务统计资源占用， 并且也不会在 **systemd-cgtop** 中显示详细的资源占用数据。

选项 (OPTIONS)[¶](#%E9%80%89%E9%A1%B9(OPTIONS) "Permalink to this headline")
--------------------------------------------------------------------------

可以识别的选项如下：

`-p`, `--order=path`[¶](#-p "Permalink to this term")

按照控制组的路径 排序

`-t`, `--order=tasks`[¶](#-t "Permalink to this term")

按照控制组内的任务 / 进程数量排序

`-c`, `--order=cpu`[¶](#-c "Permalink to this term")

按照 CPU 占用率排序

`-m`, `--order=memory`[¶](#-m "Permalink to this term")

按照内存使用量排序

`-i`, `--order=io`[¶](#-i "Permalink to this term")

按照磁盘吞吐率排序

`-b`, `--batch`[¶](#-b "Permalink to this term")

以 "批量" 模式运行。也就是，不接受任何输入， 一直运行到 `--iterations=` 刷新次数结束或者运行中途被杀死。 此模式可以用于将 **systemd-cgtop** 的输出 发送给另一个程序或写入某个文件。

`-r`, `--raw`[¶](#-r "Permalink to this term")

将字节计数 (内存占用与 I/O 度量) 按照原始数值显示， 而不是按照人类易读的方式显示。

`--cpu=percentage`, `--cpu=time`[¶](#--cpu=percentage "Permalink to this term")

控制 CPU 占用 是按照百分比显示还是按照总时长显示。 默认是按照百分比显示。 可以在运行时使用 **%** 按键切换显示方式。

`-P`[¶](#-P "Permalink to this term")

仅统计用户空间进程(也就是不统计内核任务)。 默认是 分别统计每一个内核线程与每一个用户空间线程。 使用此选项之后， 内核线程将会从统计中排除， 并且每一个用户空间进程 (无论内部包含了多少个线程) 都仅作为一个整体进行统计。 可以在运行时使用 **P** 按键切换统计方式。 此选项不可与 `-k` 选项一起使用。

`-k`[¶](#-k "Permalink to this term")

仅统计用户空间进程与内核线程。 默认是 分别统计每一个内核线程与每一个用户空间线程。 使用此选项之后， 统计中将会包含内核线程， 并且每一个用户空间进程 (无论内部包含了多少个线程) 都仅作为一个整体进行统计。 可以在运行时使用 **k** 按键切换统计方式。 此选项不可与 `-P` 选项一起使用。

`--recursive=`[¶](#--recursive= "Permalink to this term")

在统计控制组内的进程数量时， 是否递归的包含所有子控制组内的进程。 接受一个布尔值。 默认值 "`yes`" 表示包含。 而设为 "`no`" 则表示不包含。 可以在运行时使用 **r** 按键切换统计方式。 注意， 此选项仅影响控制组内的进程数量的统计 (也就是使用了 `-P` 或 `-k` 选项)， 而不会影响对所有任务数量的统计 (永远以递归方式统计)。

`-n`, `--iterations=`[¶](#-n "Permalink to this term")

仅刷新指定的次数， 然后退出。 设为 0 表示无限刷新。

`-1`[¶](#-1 "Permalink to this term")

`--iterations=1` 的快捷方式

`-d`, `--delay=`[¶](#-d "Permalink to this term")

指定每次刷新的时间间隔 (默认单位是秒，但是也可以使用 "`ms`","`us`","`min`" 单位后缀)。 可以在运行时使用 **+** 与 **-** 按键调整此值。

`--depth=`[¶](#--depth= "Permalink to this term")

设置 **systemd-cgtop** 遍历控制组层次树的最大深度。 设为 0 表示仅监视根控制组， 设为 1 表示仅监视第一层控制组，以此类推。 默认值是 3 层。

``-M _`MACHINE`_``, ``--machine=_`MACHINE`_``[¶](#-M%20MACHINE "Permalink to this term")

限制仅显示 与 _`MACHINE`_ 容器对应的控制组。 当明确指定了控制组路径的时候，不可以使用此选项。

`-h`, `--help`[¶](#-h "Permalink to this term")

显示简短的帮助信息并退出。

`--version`[¶](#--version "Permalink to this term")

显示简短的版本信息并退出。

按键 [¶](#%E6%8C%89%E9%94%AE "Permalink to this headline")
--------------------------------------------------------

**systemd-cgtop** 是一个交互式工具， 用户可以通过下列按键控制它：

**h**[¶](#h "Permalink to this term")

显示简短的帮助信息

**Space**[¶](# "Permalink to this term")

立即刷新

**q**[¶](#q "Permalink to this term")

退出

**p**, **t**, **c**, **m**, **i**[¶](#p "Permalink to this term")

分别按照：路径, 任务数量, CPU 占用, 内存占用, I/O 负载 对控制组进行排序。 此设置还可以通过 `--order=` 选项 进行控制。

**%**[¶](#% "Permalink to this term")

在显示 CPU 占用的两种不同方式 (总时长, 百分比) 之间切换。 此设置还可以通过 `--cpu=` 选项进行控制。

**+**, **-**[¶](#+ "Permalink to this term")

增加 / 减少每次刷新之间的时间间隔。 此设置还可以通过 `--delay=` 选项 进行控制。

**P**[¶](#P "Permalink to this term")

在统计全部任务、仅统计用户空间进程之间进行切换。 此设置还可以通过 `-P` 选项 进行控制。

**k**[¶](#k "Permalink to this term")

在统计全部任务、仅统计用户空间进程与内核线程之间进行切换。 此设置还可以通过 `-k` 选项 进行控制。

**r**[¶](#r "Permalink to this term")

在两种统计控制组内进程数量的方式上进行切换： (1) 递归的包含子控制组内的进程；(2) 不包含任何子控制组内的进程。 此设置还可以通过 `--recursive=` 选项进行控制。 当统计所有任务时，此按键不可用。 此按键仅用于进程的统计 (也就是 **P** 或 **k** 按键)。

退出状态 [¶](#%E9%80%80%E5%87%BA%E7%8A%B6%E6%80%81 "Permalink to this headline")
----------------------------------------------------------------------------

返回值为 0 表示成功， 非零返回值表示失败代码。