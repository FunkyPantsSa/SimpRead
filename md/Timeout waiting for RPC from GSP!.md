> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [github.com](https://github.com/NVIDIA/open-gpu-kernel-modules/issues/446)

### NVIDIA Open GPU Kernel Modules Version  
NVIDIA 开放 GPU 内核模块版本

525.85.05

### Does this happen with the proprietary driver (of the same version) as well?  
这是否也发生在专有驱动程序（相同版本）上？

I cannot test this  
我无法测试这个

### Operating System and Version  
操作系统和版本

Arch Linux

### Kernel Release 内核发布

Linux [HOSTNAME] 6.1.8-arch1-1 [#1](https://github.com/NVIDIA/open-gpu-kernel-modules/issues/1) SMP PREEMPT_DYNAMIC Tue, 24 Jan 2023 21:07:04 +0000 x86_64 GNU/Linux  
Linux [主机名] 6.1.8-arch1-1 #1 SMP PREEMPT_DYNAMIC 2023 年 1 月 24 日星期二 21:07:04 +0000 x86_64 GNU/Linux

### Hardware: GPU 硬件：GPU

GPU 0: NVIDIA GeForce RTX 3050 Laptop GPU (UUID: GPU-071149ae-386e-0017-3b5b-7ea80801f725)  
GPU 0: NVIDIA GeForce RTX 3050 笔记本 GPU (UUID: GPU-071149ae-386e-0017-3b5b-7ea80801f725)

### Describe the bug 描述错误

When I open a OpenGL application, like Yamagi Quake II, at a certain point the whole system freezes, and run in like 1 FPS per second. I generally have to REISUB when this happens.  
当我打开一个 OpenGL 应用程序，比如 Yamagi Quake II，到一定程度时整个系统会冻结，每秒只能运行 1 帧。通常在这种情况下我不得不使用 REISUB。

### To Reproduce 复制

1.  Open Yamagi Quake II  
    打开 Yamagi Quake II
2.  Change workspace, open pavucontrol to select a new audio sink for the game, switch back  
    更改工作区，打开 pavucontrol 以选择游戏的新音频输出设备，然后切换回去

### Bug Incidence 错误发生率

Always 始终如一

### nvidia-bug-report.log.gz

[nvidia-bug-report.log.gz](https://github.com/NVIDIA/open-gpu-kernel-modules/files/10510199/nvidia-bug-report.log.gz)

### More Info 更多信息

Related: [#272](https://github.com/NVIDIA/open-gpu-kernel-modules/issues/272) 相关：#272

 

This looks a lot like nvbug 3806304.  
这看起来很像 nvbug 3806304。

 

I met this on `525.85.12` for A30.  
我在 `525.85.12` 上遇到了这个 A30。

 

This issue seems to exists on `525.60.13` for **A40** and **A100** as well. Please fix ASAP!  
这个问题似乎也存在于 A40 和 A100 上。请尽快修复！  
[bug-nv-smi.txt 错误 - nv-smi.txt](https://github.com/NVIDIA/open-gpu-kernel-modules/files/10832170/bug-nv-smi.txt)  
[dmesg.txt](https://github.com/NVIDIA/open-gpu-kernel-modules/files/10832171/dmesg.txt)

 

Hi [@aritger](https://github.com/aritger), is there any solution about this?  
嗨，有关于这个的解决方案吗？

 

I think "Timeout waiting for RPC from GSP!" is a pretty generic symptom, with many possible causes. The reproduction steps that lead up to it will matter to help distinguish different bugs, as will the other NVRM dmesg spew around it.  
我认为 “等待 GSP 的 RPC 超时！” 是一个相当普遍的症状，可能有很多可能的原因。导致这个问题的重现步骤将有助于区分不同的错误，与之相关的其他 NVRM dmesg 输出也很重要。

I don't have specific reason to expect it is already fixed, but it may be worth testing the most recent 525.89.02 driver. 530.xx drivers will hopefully be released soon, and they will have a lot of changes relative to 525.xx, so that will also be worth testing.  
我没有具体的理由期望它已经修复了，但是测试最新的 525.89.02 驱动程序可能是值得的。530.xx 驱动程序很快就会发布，相对于 525.xx，它们会有很多变化，所以也值得测试。

Beyond that, if you see similar "Timeout waiting for RPC from GSP!" messages, it is worth attaching a complete nvidia-bug-report.log.gz, and describing the steps that led to it, so that we can compare instances of the symptom.  
如果您看到类似的 “等待 GSP 的 RPC 超时！” 的消息，值得附上完整的 nvidia-bug-report.log.gz，并描述导致此问题的步骤，以便我们可以比较症状的实例。

 

Thanks [@aritger](https://github.com/aritger)，[nvidia-bug-report.log.gz](https://github.com/NVIDIA/open-gpu-kernel-modules/files/10836852/nvidia-bug-report.log.gz).  
谢谢，nvidia-bug-report.log.gz。

The problem occurs after running in the kubernetes environment for a period of time, and `nvidia-smi` will get stuck for a while. The specific error phenomenon is similar to [@jelmd](https://github.com/jelmd) [#446 (comment)](https://github.com/NVIDIA/open-gpu-kernel-modules/issues/446#issuecomment-1445190598). FYI, I got some help from nvidia-docker community([NVIDIA/nvidia-docker#1648 (comment)](https://github.com/NVIDIA/nvidia-docker/issues/1648#issuecomment-1441139460)), but not sure if the root cause of this problem is driver or NVIDIA-docker related.  
在 Kubernetes 环境中运行一段时间后出现问题， `nvidia-smi` 会卡住一段时间。具体的错误现象类似于＃446（评论）。我从 nvidia-docker 社区得到了一些帮助（NVIDIA/nvidia-docker#1648（评论）），但不确定这个问题的根本原因是驱动程序还是与 NVIDIA-docker 相关的。

 

FWIIW: We do not use any nvidia-docker or similar bloat, just plain lxc and passthrough the devices to the related zones alias containers as needed. So IMHO the nvidia-container-toolkit is not really related to the problem.  
我们不使用任何 nvidia-docker 或类似的臃肿工具，只使用纯粹的 lxc，并根据需要将设备直通到相关的区域别名容器。所以在我看来，nvidia-container-toolkit 与这个问题并没有真正的关联。

 

Happend again on another machine:  
又在另一台机器上发生了

```
...
[  +0.000094]  ? _nv011159rm+0x62/0x2e0 [nvidia]
[  +0.000090]  ? _nv039897rm+0xdb/0x140 [nvidia]
[  +0.000073]  ? _nv041022rm+0x2ce/0x3a0 [nvidia]
[  +0.000103]  ? _nv015438rm+0x788/0x800 [nvidia]
[  +0.000064]  ? _nv039416rm+0xac/0xe0 [nvidia]
[  +0.000092]  ? _nv041024rm+0xac/0x140 [nvidia]
[  +0.000095]  ? _nv041023rm+0x37a/0x4d0 [nvidia]
[  +0.000070]  ? _nv039319rm+0xc9/0x150 [nvidia]
[  +0.000151]  ? _nv039320rm+0x42/0x70 [nvidia]
[  +0.000180]  ? _nv000552rm+0x49/0x60 [nvidia]
[  +0.000219]  ? _nv000694rm+0x7fb/0xc80 [nvidia]
[  +0.000195]  ? rm_ioctl+0x54/0xb0 [nvidia]
[  +0.000132]  ? nvidia_ioctl+0x6e3/0x850 [nvidia]
[  +0.000003]  ? get_max_files+0x20/0x20
[  +0.000134]  ? nvidia_frontend_unlocked_ioctl+0x3b/0x50 [nvidia]
[  +0.000002]  ? do_vfs_ioctl+0x407/0x670
[  +0.000003]  ? __secure_computing+0xa4/0x110
[  +0.000002]  ? ksys_ioctl+0x67/0x90
[  +0.000002]  ? __x64_sys_ioctl+0x1a/0x20
[  +0.000002]  ? do_syscall_64+0x57/0x190
[  +0.000002]  ? entry_SYSCALL_64_after_hwframe+0x5c/0xc1
[  +6.010643] NVRM: Xid (PCI:0000:83:00): 119, pid=2710030, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 10 (FREE) (0xa55a0050 0x0).
...
Mon Feb 27 16:08:33 2023       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 525.60.13    Driver Version: 525.60.13    CUDA Version: 12.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA A40          On   | 00000000:03:00.0 Off |                    0 |
|  0%   24C    P8    14W / 300W |      0MiB / 46068MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   1  NVIDIA A40          On   | 00000000:04:00.0 Off |                    0 |
|  0%   25C    P8    13W / 300W |      0MiB / 46068MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   2  NVIDIA A40          On   | 00000000:43:00.0 Off |                    0 |
|  0%   40C    P0    81W / 300W |    821MiB / 46068MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   3  NVIDIA A40          On   | 00000000:44:00.0 Off |                    0 |
|  0%   30C    P8    14W / 300W |      0MiB / 46068MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   4  ERR!                On   | 00000000:83:00.0 Off |                 ERR! |
|ERR!  ERR! ERR!    ERR! / ERR! |      0MiB / 46068MiB |    ERR!      Default |
|                               |                      |                 ERR! |
+-------------------------------+----------------------+----------------------+
|   5  NVIDIA A40          On   | 00000000:84:00.0 Off |                    0 |
|  0%   23C    P8    14W / 300W |      3MiB / 46068MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   6  NVIDIA A40          On   | 00000000:C3:00.0 Off |                    0 |
|  0%   22C    P8    15W / 300W |      3MiB / 46068MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   7  NVIDIA A40          On   | 00000000:C4:00.0 Off |                    0 |
|  0%   21C    P8    14W / 300W |      3MiB / 46068MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    2   N/A  N/A   2111354      C   ...eratornet/venv/bin/python      818MiB |
+-----------------------------------------------------------------------------+


```

 

[@jelmd](https://github.com/jelmd) +1, I ran into this problem again. Hi [@aritger](https://github.com/aritger) [@Joshua-Ashton](https://github.com/Joshua-Ashton) maybe this is a Driver issue, please take a look.  
+1，我又遇到了这个问题。嗨，也许这是一个驱动程序问题，请看一下。

```
Fri Mar  3 14:41:55 2023       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 525.85.12    Driver Version: 525.85.12    CUDA Version: 12.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA A30          Off  | 00000000:01:00.0 Off |                    0 |
| N/A   26C    P0    29W / 165W |      0MiB / 24576MiB |      0%      Default |
|                               |                      |             Disabled |
+-------------------------------+----------------------+----------------------+
|   1  NVIDIA A30          Off  | 00000000:22:00.0 Off |                    0 |
| N/A   40C    P0    92W / 165W |  16096MiB / 24576MiB |     88%      Default |
|                               |                      |             Disabled |
+-------------------------------+----------------------+----------------------+
|   2  NVIDIA A30          Off  | 00000000:41:00.0 Off |                    0 |
| N/A   42C    P0   141W / 165W |  14992MiB / 24576MiB |     33%      Default |
|                               |                      |             Disabled |
+-------------------------------+----------------------+----------------------+
|   3  ERR!                Off  | 00000000:61:00.0 Off |                 ERR! |
|ERR!  ERR! ERR!    ERR! / ERR! |  23025MiB / 24576MiB |    ERR!      Default |
|                               |                      |                 ERR! |
+-------------------------------+----------------------+----------------------+
|   4  NVIDIA A30          Off  | 00000000:81:00.0 Off |                    0 |
| N/A   26C    P0    26W / 165W |      0MiB / 24576MiB |      0%      Default |
|                               |                      |             Disabled |
+-------------------------------+----------------------+----------------------+
|   5  NVIDIA A30          Off  | 00000000:A1:00.0 Off |                    0 |
| N/A   26C    P0    28W / 165W |      0MiB / 24576MiB |      0%      Default |
|                               |                      |             Disabled |
+-------------------------------+----------------------+----------------------+
|   6  NVIDIA A30          Off  | 00000000:C1:00.0 Off |                    0 |
| N/A   25C    P0    29W / 165W |      0MiB / 24576MiB |      0%      Default |
|                               |                      |             Disabled |
+-------------------------------+----------------------+----------------------+
|   7  NVIDIA A30          Off  | 00000000:E1:00.0 Off |                    0 |
| N/A   24C    P0    25W / 165W |      0MiB / 24576MiB |      0%      Default |
|                               |                      |             Disabled |
+-------------------------------+----------------------+----------------------+
                                                                               


```

```
[Fri Mar  3 04:23:22 2023] NVRM: GPU at PCI:0000:61:00: GPU-e59ce3f9-af53-a0dd-1d2c-8beaa74aa635
[Fri Mar  3 04:23:22 2023] NVRM: GPU Board Serial Number: 1322621149782
[Fri Mar  3 04:23:22 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1344368, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 10 (FREE) (0xa55a00a0 0x0).
[Fri Mar  3 04:23:22 2023] CPU: 72 PID: 1344368 Comm: nvidia-smi Tainted: P           OE     5.10.0-20-amd64 #1 Debian 5.10.158-2
[Fri Mar  3 04:23:22 2023] Hardware name: Inspur NF5468A5/YZMB-02382-101, BIOS 4.02.12 01/28/2022
[Fri Mar  3 04:23:22 2023] Call Trace:
[Fri Mar  3 04:23:22 2023]  dump_stack+0x6b/0x83
[Fri Mar  3 04:23:22 2023]  _nv011231rm+0x39d/0x470 [nvidia]
[Fri Mar  3 04:23:22 2023]  ? _nv011168rm+0x62/0x2e0 [nvidia]
[Fri Mar  3 04:23:22 2023]  ? _nv040022rm+0xdb/0x140 [nvidia]
[Fri Mar  3 04:23:22 2023]  ? _nv041148rm+0x2ce/0x3a0 [nvidia]
[Fri Mar  3 04:23:22 2023]  ? _nv015451rm+0x788/0x800 [nvidia]
[Fri Mar  3 04:23:22 2023]  ? _nv039541rm+0xac/0xe0 [nvidia]
[Fri Mar  3 04:23:22 2023]  ? _nv041150rm+0xac/0x140 [nvidia]
[Fri Mar  3 04:23:22 2023]  ? _nv041149rm+0x37a/0x4d0 [nvidia]
[Fri Mar  3 04:23:22 2023]  ? _nv039443rm+0xc9/0x150 [nvidia]
[Fri Mar  3 04:23:22 2023]  ? _nv039444rm+0x42/0x70 [nvidia]
[Fri Mar  3 04:23:22 2023]  ? _nv000554rm+0x49/0x60 [nvidia]
[Fri Mar  3 04:23:22 2023]  ? _nv000694rm+0x7fb/0xc80 [nvidia]
[Fri Mar  3 04:23:22 2023]  ? rm_ioctl+0x54/0xb0 [nvidia]
[Fri Mar  3 04:23:22 2023]  ? nvidia_ioctl+0x6cd/0x830 [nvidia]
[Fri Mar  3 04:23:22 2023]  ? nvidia_frontend_unlocked_ioctl+0x37/0x50 [nvidia]
[Fri Mar  3 04:23:22 2023]  ? __x64_sys_ioctl+0x8b/0xc0
[Fri Mar  3 04:23:22 2023]  ? do_syscall_64+0x33/0x80
[Fri Mar  3 04:23:22 2023]  ? entry_SYSCALL_64_after_hwframe+0x61/0xc6
[Fri Mar  3 04:24:07 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1344368, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 10 (FREE) (0xa55a0030 0x0).
[Fri Mar  3 04:24:52 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1344368, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 10 (FREE) (0xa55a0020 0x0).
[Fri Mar  3 04:25:38 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20801348 0x410).
[Fri Mar  3 04:26:23 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 103 (GSP_RM_ALLOC) (0x0 0x6c).
[Fri Mar  3 04:27:08 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 103 (GSP_RM_ALLOC) (0x80 0x38).
[Fri Mar  3 04:27:53 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 103 (GSP_RM_ALLOC) (0x2080 0x4).
[Fri Mar  3 04:28:38 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20800110 0x84).
[Fri Mar  3 04:29:23 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080014b 0x5).
[Fri Mar  3 04:30:08 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20800157 0x0).
[Fri Mar  3 04:30:53 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080014b 0x5).
[Fri Mar  3 04:31:38 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080014b 0x5).
[Fri Mar  3 04:32:23 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080014b 0x5).
[Fri Mar  3 04:33:08 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080014b 0x5).
[Fri Mar  3 04:33:53 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080852e 0x208).
[Fri Mar  3 04:34:38 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20808513 0x598).
[Fri Mar  3 04:35:23 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080a068 0x4).
[Fri Mar  3 04:36:03 2023] INFO: task nvidia-smi:1346229 blocked for more than 120 seconds.
[Fri Mar  3 04:36:03 2023]       Tainted: P           OE     5.10.0-20-amd64 #1 Debian 5.10.158-2
[Fri Mar  3 04:36:03 2023] "echo 0 > /proc/sys/kernel/hung_task_timeout_secs" disables this message.
[Fri Mar  3 04:36:03 2023] task:nvidia-smi      state:D stack:    0 pid:1346229 ppid:1346228 flags:0x00000000
[Fri Mar  3 04:36:03 2023] Call Trace:
[Fri Mar  3 04:36:03 2023]  __schedule+0x282/0x880
[Fri Mar  3 04:36:03 2023]  ? rwsem_spin_on_owner+0x74/0xd0
[Fri Mar  3 04:36:03 2023]  schedule+0x46/0xb0
[Fri Mar  3 04:36:03 2023]  rwsem_down_write_slowpath+0x246/0x4d0
[Fri Mar  3 04:36:03 2023]  os_acquire_rwlock_write+0x31/0x40 [nvidia]
[Fri Mar  3 04:36:03 2023]  _nv038505rm+0xc/0x30 [nvidia]
[Fri Mar  3 04:36:03 2023]  ? _nv039453rm+0x18d/0x1d0 [nvidia]
[Fri Mar  3 04:36:03 2023]  ? _nv041182rm+0x45/0xd0 [nvidia]
[Fri Mar  3 04:36:03 2023]  ? _nv041127rm+0x142/0x2b0 [nvidia]
[Fri Mar  3 04:36:03 2023]  ? _nv039415rm+0x15a/0x2e0 [nvidia]
[Fri Mar  3 04:36:03 2023]  ? _nv039416rm+0x5b/0x90 [nvidia]
[Fri Mar  3 04:36:03 2023]  ? _nv039416rm+0x31/0x90 [nvidia]
[Fri Mar  3 04:36:03 2023]  ? _nv000559rm+0x5a/0x70 [nvidia]
[Fri Mar  3 04:36:03 2023]  ? _nv000559rm+0x33/0x70 [nvidia]
[Fri Mar  3 04:36:03 2023]  ? _nv000694rm+0x94a/0xc80 [nvidia]
[Fri Mar  3 04:36:03 2023]  ? rm_ioctl+0x54/0xb0 [nvidia]
[Fri Mar  3 04:36:03 2023]  ? nvidia_ioctl+0x6cd/0x830 [nvidia]
[Fri Mar  3 04:36:03 2023]  ? nvidia_frontend_unlocked_ioctl+0x37/0x50 [nvidia]
[Fri Mar  3 04:36:03 2023]  ? __x64_sys_ioctl+0x8b/0xc0
[Fri Mar  3 04:36:03 2023]  ? do_syscall_64+0x33/0x80
[Fri Mar  3 04:36:03 2023]  ? entry_SYSCALL_64_after_hwframe+0x61/0xc6
[Fri Mar  3 04:36:08 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080a618 0x181c).
[Fri Mar  3 04:36:53 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080a612 0xd98).
[Fri Mar  3 04:37:38 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20809009 0x8).
[Fri Mar  3 04:38:23 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20800a4c 0x4).
[Fri Mar  3 04:39:08 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20800a4c 0x4).
[Fri Mar  3 04:39:53 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20800a4c 0x4).
[Fri Mar  3 04:40:38 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 103 (GSP_RM_ALLOC) (0x208f 0x0).
[Fri Mar  3 04:41:23 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x208f1105 0x8).
[Fri Mar  3 04:42:08 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 10 (FREE) (0xa55a00a0 0x0).
[Fri Mar  3 04:42:53 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 10 (FREE) (0xa55a0030 0x0).
[Fri Mar  3 04:43:38 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1346198, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 10 (FREE) (0xa55a0020 0x0).
[Fri Mar  3 04:44:24 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20801348 0x410).
[Fri Mar  3 04:45:09 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 103 (GSP_RM_ALLOC) (0x0 0x6c).
[Fri Mar  3 04:45:54 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 103 (GSP_RM_ALLOC) (0x80 0x38).
[Fri Mar  3 04:46:39 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 103 (GSP_RM_ALLOC) (0x2080 0x4).
[Fri Mar  3 04:47:24 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20800110 0x84).
[Fri Mar  3 04:48:09 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080014b 0x5).
[Fri Mar  3 04:48:54 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20800157 0x0).
[Fri Mar  3 04:49:39 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080014b 0x5).
[Fri Mar  3 04:50:24 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080014b 0x5).
[Fri Mar  3 04:51:09 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080014b 0x5).
[Fri Mar  3 04:51:54 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080014b 0x5).
[Fri Mar  3 04:52:39 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080852e 0x208).
[Fri Mar  3 04:53:24 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20808513 0x598).
[Fri Mar  3 04:54:09 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080a068 0x4).
[Fri Mar  3 04:54:54 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080a618 0x181c).
[Fri Mar  3 04:55:39 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080a612 0xd98).
[Fri Mar  3 04:56:24 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20809009 0x8).
[Fri Mar  3 04:57:09 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20800a4c 0x4).
[Fri Mar  3 04:57:54 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20800a4c 0x4).
[Fri Mar  3 04:58:39 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20800a4c 0x4).
[Fri Mar  3 04:59:24 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 103 (GSP_RM_ALLOC) (0x208f 0x0).
[Fri Mar  3 05:00:09 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x208f1105 0x8).
[Fri Mar  3 05:00:54 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 10 (FREE) (0xa55a00a0 0x0).
[Fri Mar  3 05:01:39 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 10 (FREE) (0xa55a0030 0x0).
[Fri Mar  3 05:02:24 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1365232, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 10 (FREE) (0xa55a0020 0x0).
[Fri Mar  3 05:03:11 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1385412, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20801348 0x410).
[Fri Mar  3 05:03:56 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1385412, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 103 (GSP_RM_ALLOC) (0x0 0x6c).
[Fri Mar  3 05:04:41 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1385412, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 103 (GSP_RM_ALLOC) (0x80 0x38).
[Fri Mar  3 05:05:26 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1385412, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 103 (GSP_RM_ALLOC) (0x2080 0x4).
[Fri Mar  3 05:06:11 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1385412, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20800110 0x84).
[Fri Mar  3 05:06:56 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1385412, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x2080014b 0x5).
[Fri Mar  3 05:07:41 2023] NVRM: Xid (PCI:0000:61:00): 119, pid=1385412, name=nvidia-smi, Timeout waiting for RPC from GSP! Expected function 76 (GSP_RM_CONTROL) (0x20800157 0x0).


```

 

Also happening here on a A100-PCIE-40GB using driver 530.30.02 and CUDA 12.1.  
也发生在使用驱动程序 530.30.02 和 CUDA 12.1 的 A100-PCIE-40GB 上。

 

Hi [@lpla](https://github.com/lpla) , what's your use case environment? In kubernetes?  
你好，你的使用环境是什么？在 Kubernetes 中吗？

 

There is no environment. It triggered the bug several times in both 525 and 530 driver. It is a Machine Learning inference command line written in PyTorch.  
没有环境。它在 525 和 530 驱动程序中多次触发了错误。这是一个用 PyTorch 编写的机器学习推理命令行。

 

> There is no environment. It triggered the bug several times in both 525 and 530 driver. It is a Machine Learning inference command line written in PyTorch.  
> 没有环境。它在 525 和 530 驱动程序中多次触发了错误。这是一个用 PyTorch 编写的机器学习推理命令行。

Have you tried the 520.* driver ? Is it work?  
你试过 520.* 驱动程序吗？它能工作吗？

 

FWIW: Most of our users use PyTorch as well. Perhaps it tortures GPUs too hard ;-)  
就我所知：我们大多数用户也使用 PyTorch。也许它对 GPU 的压力太大了;-)

 

We also use PyTorch on the GPUs, but the 470 driver used before has been more stable.  
我们也在 GPU 上使用 PyTorch，但之前使用的 470 驱动更加稳定。

 

Yepp. np with 470, too.  
没问题，470 也可以。

 

> > There is no environment. It triggered the bug several times in both 525 and 530 driver. It is a Machine Learning inference command line written in PyTorch.  
> > 没有环境。它在 525 和 530 驱动程序中多次触发了错误。这是一个用 PyTorch 编写的机器学习推理命令行。
> 
> Have you tried the 520.* driver ? Is it work?  
> 你试过 520.* 驱动程序吗？它能工作吗？

That's my next test. In fact, that's exactly the version I was using before upgrading from Ubuntu 20.04 with kernel 5.15 and driver 520 to Ubuntu 22.04 with kernel 5.19 and driver 525 last month. It was working perfect with that previous setup.  
这是我的下一个测试。事实上，这正是我在上个月从 Ubuntu 20.04 升级到 Ubuntu 22.04 时使用的版本，当时的内核版本是 5.15，驱动版本是 520。在那个之前的设置下，它运行得非常完美。

 

Same thing on another machine. FWIW: I removed `/usr/lib/firmware/nvidia/525.60.13` - perhaps this fixes the problem.  
在另一台机器上也是同样的情况。顺便说一句：我移除了 `/usr/lib/firmware/nvidia/525.60.13` - 或许这样可以解决问题。

 

### UPDATE 更新

They have confirmed `Xid 119` this bug. They said that the `GSP` feature was introduced from version 510, but it has not been fixed yet. They only gave the method of disabling it mentioned below or suggested that we downgrade the version to <510(e.g. 470) so that it is more stable.  
他们已经确认了这个错误。他们说这个功能是从 510 版本开始引入的，但是还没有修复。他们只提供了下面提到的禁用它的方法，或者建议我们降级到 510 以下的版本（例如 470），这样更稳定。

Hi [@jelmd](https://github.com/jelmd) [@lpla](https://github.com/lpla) , as the NVIDIA customer, we communicate with the NVIDIA support team today and according to the `nvidia-bug-report.log.gz` they advised us to disable `GSP-RM`.  
嗨，作为 NVIDIA 的客户，我们今天与 NVIDIA 支持团队进行了沟通，并根据他们的建议禁用了 `GSP-RM` 。

1.  To disable GSP-RM: 禁用 GSP-RM：

```
sudo su -c 'echo options nvidia NVreg_EnableGpuFirmware=0 > /etc/modprobe.d/nvidia-gsp.conf'


```

2.  Enable the kernel 启用内核

```
# if ubuntu
sudo update-initramfs -u

# if centos
dracut -f 


```

3.  Reboot 重启
4.  Check if work. If `EnableGpuFirmware: 0` then `GSP-RM` is disabled.  
    检查工作是否正常。如果 `EnableGpuFirmware: 0` ，则 `GSP-RM` 被禁用。

```
cat /proc/driver/nvidia/params | grep EnableGpuFirmware


```

Since our problem node still has tasks running, I haven't tried it yet, I will try this method tonight or tomorrow morning, just for reference. :)  
由于我们的问题节点仍在运行任务，我还没有尝试过，我会在今晚或明天早上尝试这种方法，仅供参考。 :)

 

I'm also seeing XID 119s on some 510 drivers. Have not tried 525 or 520.  
我也在一些 510 驱动程序上看到了 XID 119s。还没有尝试过 525 或 520。

 

Driver 525.60.13 with A40 GSP disable.But nvidia-but-report also have GSP timeout error.  
驱动程序 525.60.13 禁用了 A40 GSP。但是 nvidia-but-report 也报告了 GSP 超时错误。

 

Hi [@stephenroller](https://github.com/stephenroller) [@liming5619](https://github.com/liming5619) , maybe it's better to downgrade the driver version. On one hand, the GSP feature was introduced by NVIDIA since 510 but has not been fixed yet. On the other hand, 470 is an [LTS version](https://docs.nvidia.com/datacenter/tesla/drivers/index.html#:~:text=Term%20Support%20Branch-,Long%20Term%20Support%20Branch,-Production%20Branch) and has been running stably in our production environment for a long time. I have already downgraded the driver of the problematic node to 470.82.01 to match our other production nodes, just for your reference. :)  
嗨，也许降级驱动程序版本会更好。一方面，GSP 功能是 NVIDIA 自 510 版本引入的，但尚未修复。另一方面，470 是一个 LTS 版本，在我们的生产环境中长时间稳定运行。我已经将有问题的节点的驱动程序降级到 470.82.01，以与我们其他的生产节点保持一致，仅供参考。 :)

 

So far disabling GSP seems to have mitigated, but maybe I've just been lucky since. Will report back if I see counter-evidence.  
到目前为止，禁用 GSP 似乎已经缓解了，但也许我只是运气好。如果我看到反证，我会回报的。

 

Yepp, removing /usr/lib/firmware/nvidia/5xx.* seems to fix the problem, too (did not use NVreg_EnableGpuFirmware=0).  
是的，删除 /usr/lib/firmware/nvidia/5xx.* 似乎也可以解决问题（没有使用 NVreg_EnableGpuFirmware=0）。

 

> ### UPDATE 更新
> 
> They have confirmed `Xid 119` this bug. They said that the `GSP` feature was introduced from version 510, but it has not been fixed yet. They only gave the method of disabling it mentioned below or suggested that we downgrade the version to <510(e.g. 470) so that it is more stable.  
> 他们已经确认了这个错误。他们说这个功能是从 510 版本开始引入的，但是还没有修复。他们只提供了下面提到的禁用它的方法，或者建议我们降级到 510 以下的版本（例如 470），这样更稳定。
> 
> Hi [@jelmd](https://github.com/jelmd) [@lpla](https://github.com/lpla) , as the NVIDIA customer, we communicate with the NVIDIA support team today and according to the `nvidia-bug-report.log.gz` they advised us to disable `GSP-RM`.  
> 嗨，作为 NVIDIA 的客户，我们今天与 NVIDIA 支持团队进行了沟通，并根据他们的建议禁用了 `GSP-RM` 。
> 
> ```
> 1. To disable GSP-RM:
> 
> 
> ```
> 
> ```
> sudo su -c 'echo options nvidia NVreg_EnableGpuFirmware=0 > /etc/modprobe.d/nvidia-gsp.conf'
> 
> 
> ```
> 
> ```
> 2. Enable the kernel
> 
> 
> ```
> 
> ```
> # if ubuntu
> sudo update-initramfs -u
> 
> # if centos
> dracut -f 
> 
> 
> ```
> 
> ```
> 3. Reboot
> 
> 4. Check if work. If `EnableGpuFirmware: 0` then `GSP-RM` is disabled.
> 
> 
> ```
> 
> ```
> cat /proc/driver/nvidia/params | grep EnableGpuFirmware
> 
> 
> ```
> 
> Since our problem node still has tasks running, I haven't tried it yet, I will try this method tonight or tomorrow morning, just for reference. :)  
> 由于我们的问题节点仍在运行任务，我还没有尝试过，我会在今晚或明天早上尝试这种方法，仅供参考。 :)

We disable it on 2 hosts - 8x A100 GPUs, If this workaround will work, I will also give a feedback.  
我们在两台主机上禁用了它 - 8 个 A100 GPU。如果这个解决方法有效，我会给出反馈。

 

Feedback: After a week, I can say all servers with the A100 boards are running stable after we disabled the GSP. No GPU crashes anymore.  
反馈：经过一周的时间，我可以说所有使用 A100 主板的服务器在我们禁用了 GSP 后都运行稳定。不再出现 GPU 崩溃。

[@fighterhit](https://github.com/fighterhit) thank you for sharing the workaround with us.  
感谢您与我们分享解决方法。

 

I have a similar issue, after disabling GSP, It took more than 5 minutes to give output as "True".  
我有一个类似的问题，在禁用 GSP 后，花了超过 5 分钟才输出 "True"。

```
# cat /etc/modprobe.d/nvidia-gsp.conf
options nvidia NVreg_EnableGpuFirmware=0
# cat /proc/driver/nvidia/params | grep EnableGpuFirmware
EnableGpuFirmware: 0
EnableGpuFirmwareLogs: 2


```

Strange thing is, I'm booting up vms with images which has GPU driver pre-installed, on a host with 4 cards, 2 out of 4 cards ends up with a similar issue.  
奇怪的是，我正在使用预先安装了 GPU 驱动程序的镜像启动虚拟机，但在拥有 4 张显卡的主机上，其中 2 张显卡出现了类似的问题。

Please suggest a fix as its hampering our prod environments. Please let me know if any additional commands or log output that i should provide.  
请建议一个修复方案，因为它正在影响我们的生产环境。如果需要提供任何额外的命令或日志输出，请告诉我。

We also have few requirements based on CUDA 11.8 and hence we cannot roll back to driver 470  
我们还有一些基于 CUDA 11.8 的要求，因此我们无法回滚到驱动程序 470。

 

[@mdrasheek](https://github.com/mdrasheek) it could be that the driver config inside the VMs is overriding your changes to GSP.  
可能是虚拟机内的驱动程序配置覆盖了您对 GSP 的更改。  
As it is, disabling GSP should resolve the problem. Check in your logs which xid error you are having.  
禁用 GSP 应该可以解决问题。检查日志，看看你遇到了哪个 xid 错误。

 

[@mdrasheek](https://github.com/mdrasheek) , [@fighterhit](https://github.com/fighterhit)  
Is it DGX box.  
这是 DGX 盒子吗。  
Pls check the matrix for xid errors. It could be that GPU having H/W error.  
请检查矩阵中的 xid 错误。可能是 GPU 出现了硬件错误。  
[https://docs.nvidia.com/deploy/xid-errors/index.html#topic_4](https://docs.nvidia.com/deploy/xid-errors/index.html#topic_4)

 

Same issues were raised when using docker with GPU passthrough with the following packages on NVIDIA A40  
使用 NVIDIA A40 时，使用 docker 进行 GPU 透传时出现了相同的问题

*   cuda: 12.1 CUDA：12.1
*   nvidia-driver-cuda.x86_64: 530.30.02
*   nvidia-container-toolkit.x86_64: 1.13.0-1
*   docker-ce.x86_64: 23.0.3-1.el8

 

Looks you are using 530.30.02 beta version, this version not support A40 Card.  
看起来你正在使用 530.30.02 测试版，这个版本不支持 A40 卡。  
[https://www.nvidia.com/download/driverResults.aspx/199985/en-us/](https://www.nvidia.com/download/driverResults.aspx/199985/en-us/)

Pls check version 525.105.17.  
请检查版本号 525.105.17。

 

> [@mdrasheek](https://github.com/mdrasheek) , [@fighterhit](https://github.com/fighterhit) Is it DGX box. Pls check the matrix for xid errors. It could be that GPU having H/W error. [https://docs.nvidia.com/deploy/xid-errors/index.html#topic_4](https://docs.nvidia.com/deploy/xid-errors/index.html#topic_4)  
> 这是 DGX 盒子吗？请检查矩阵中的 xid 错误。可能是 GPU 出现了硬件错误。https://docs.nvidia.com/deploy/xid-errors/index.html#topic_4

No, Its not DGX. Its A100 80 GB PCI, a VM is using it through pass through.  
不，不是 DGX。是 A100 80 GB PCI，一个虚拟机通过直通方式使用它。

 

> I'm also seeing XID 119s on some 510 drivers. Have not tried 525 or 520.  
> 我也在一些 510 驱动程序上看到了 XID 119s。还没有尝试过 525 或 520。

The same XID 119 issue for 520.105 driver.  
相同的 XID 119 问题适用于 520.105 驱动程序。

 

I still encounter this problem on 1 out of 2 A100s after upgrading drive to 530.41.03.  
我在升级驱动到 530.41.03 之后，仍然在两个 A100 中的一个上遇到了这个问题。

 

I want to highlight a point here, post enabling MIG and creating a full MIG Slice the GPU works without lag.  
我想在这里强调一点，启用 MIG 并创建一个完整的 MIG 切片后，GPU 可以无延迟地工作。

nvidia-smi -mig 1  
nvidia-smi mig -cgi 0 -C

The above commands will create slice 0 which is the entire 80GB MIG slice. But with MIG mode disabled the lag comes again. I hope this may lead a way for the resolution for some one.  
上述命令将创建切片 0，该切片是整个 80GB 的 MIG 切片。但是当 MIG 模式被禁用时，延迟问题再次出现。我希望这可能为某人提供解决方案的线索。

 

We recently got a ThinkSystem SR670-V2 with 8 A100 80GB cards. We are running 530.30.02 from yum CUDA repo in a SLURM batch. We have had these Xid 119 errors on 3 different GPUs so far running for about 3 weeks with typical TensorFlow jobs.  
我们最近获得了一台 ThinkSystem SR670-V2，配备了 8 张 A100 80GB 的显卡。我们正在使用 yum CUDA 仓库中的 530.30.02 版本，在 SLURM 批处理中运行。到目前为止，我们在运行了大约 3 周的典型 TensorFlow 作业时，已经出现了 3 个不同的 GPU 上的 Xid 119 错误。

Going to try disabling GSP as mentioned above.  
尝试禁用上述提到的 GSP。

 

> I still encounter this problem on 1 out of 2 A100s after upgrading drive to 530.41.03.  
> 我在升级驱动到 530.41.03 之后，仍然在两个 A100 中的一个上遇到了这个问题。

[@mikelou2012](https://github.com/mikelou2012) maybe have a try on 525.125.06 with GSP enable  
也许可以尝试启用 GSP 的 525.125.06 版本。

 

Hi everyone, please try with the following driver:  
大家好，请尝试使用以下驱动程序：

wget [https://us.download.nvidia.com/tesla/535.54.03/NVIDIA-Linux-x86_64-535.54.03.run](https://us.download.nvidia.com/tesla/535.54.03/NVIDIA-Linux-x86_64-535.54.03.run)

As tested from our side, It works as expected without any changes to GSP firmware and MIG in disabled state.  
经过我们的测试，没有对 GSP 固件进行任何更改，并且在禁用状态下 MIG 正常工作。

 

Unforunately ended up with similar issue in 535.54 in one of our machines.  
不幸的是，在我们的一台机器上也出现了 535.54 的类似问题。

```
Jul 11 11:42:08 e2e-85-84 kernel: [ 1412.890184] NVRM: Xid (PCI:0000:01:01): 119, pid=3395, name=python3, Timeout waiting for RPC from GSP0! Expected function 103 (GSP_RM_ALLOC) (0x80 0x38).
Jul 11 11:42:14 e2e-85-84 kernel: [ 1418.894227] NVRM: Rate limiting GSP RPC error prints for GPU at PCI:0000:01:01 (printing 1 of every 30).  The GPU likely needs to be reset.
Jul 11 11:42:38 e2e-85-84 kernel: [ 1442.922569] NVRM: Xid (PCI:0000:01:02): 119, pid=3395, name=python3, Timeout waiting for RPC from GSP1! Expected function 10 (FREE) (0x5c00000c 0x0).
Jul 11 11:42:44 e2e-85-84 kernel: [ 1448.926199] NVRM: Xid (PCI:0000:01:02): 119, pid=3395, name=python3, Timeout waiting for RPC from GSP1! Expected function 10 (FREE) (0x5c00000b 0x0).
Jul 11 11:42:50 e2e-85-84 kernel: [ 1454.930190] NVRM: Rate limiting GSP RPC error prints for GPU at PCI:0000:01:02 (printing 1 of every 30).  The GPU likely needs to be reset.```

Even after running gpu-reset, facing similar error

```Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.457563] NVRM: GPU at PCI:0000:01:02: GPU-6e4d9996-6edf-b18c-debb-269edc01a143
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.457570] NVRM: GPU Board Serial Number: 1324220003349
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.457571] NVRM: Xid (PCI:0000:01:02): 119, pid=10274, name=python3, Timeout waiting for RPC from GSP1! Expected function 76 (GSP_RM_CONTROL) (0x20803032 0x58c).
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.457585] CPU: 11 PID: 10274 Comm: python3 Tainted: P           OE     5.15.0-69-generic #76~20.04.1-Ubuntu
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.457588] Hardware name: Red Hat KVM, BIOS 1.11.0-2.el7 04/01/2014
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.457590] Call Trace:
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.457592]  
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.457595]  dump_stack_lvl+0x4a/0x63
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.457601]  dump_stack+0x10/0x16
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.457605]  os_dump_stack+0xe/0x14 [nvidia]
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.457883]  _nv011486rm+0x3ff/0x480 [nvidia]
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.458325]  ? _nv011409rm+0x5d/0x310 [nvidia]
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.458654]  ? _nv043706rm+0x4b4/0x6e0 [nvidia]
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.458996]  ? _nv033719rm+0x64/0x130 [nvidia]
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.459416]  ? _nv045949rm+0x10d/0xbe0 [nvidia]
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.459841]  ? _nv042963rm+0x1a9/0x1b0 [nvidia]
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.460154]  ? _nv044912rm+0x1f1/0x300 [nvidia]
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.460464]  ? _nv013130rm+0x335/0x630 [nvidia]
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.460745]  ? _nv043107rm+0x69/0xd0 [nvidia]
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.461030]  ? _nv011651rm+0x86/0xa0 [nvidia]
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.461305]  ? _nv000714rm+0x9c1/0xe70 [nvidia]
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.461578]  ? rm_ioctl+0x58/0xb0 [nvidia]
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.461849]  ? nvidia_ioctl+0x710/0x870 [nvidia]
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.462097]  ? do_syscall_64+0x69/0xc0
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.462102]  ? nvidia_frontend_unlocked_ioctl+0x58/0x90 [nvidia]
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.462353]  ? __x64_sys_ioctl+0x95/0xd0
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.462357]  ? do_syscall_64+0x5c/0xc0
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.462359]  ? entry_SYSCALL_64_after_hwframe+0x61/0xcb
Jul 11 12:53:46 e2e-85-84 kernel: [ 5710.462363]  
Jul 11 12:53:52 e2e-85-84 kernel: [ 5716.465188] NVRM: Xid (PCI:0000:01:01): 119, pid=10274, name=python3, Timeout waiting for RPC from GSP0! Expected function 103 (GSP_RM_ALLOC) (0x0 0x6c).
Jul 11 12:53:58 e2e-85-84 kernel: [ 5722.468809] NVRM: Xid (PCI:0000:01:01): 119, pid=10274, name=python3, Timeout waiting for RPC from GSP0! Expected function 103 (GSP_RM_ALLOC) (0x80 0x38).
Jul 11 12:54:04 e2e-85-84 kernel: [ 5728.472326] NVRM: Rate limiting GSP RPC error prints for GPU at PCI:0000:01:01 (printing 1 of every 30).  The GPU likely needs to be reset.```


```

 

Hi everyone, We have identified the issue related to our case with the help of Nvidia enterprise support team. In our machines, all the GPU cards are connected via NVLinks and as per the Nvidia doc [here](https://docs.nvidia.com/grid/13.0/grid-vgpu-user-guide/index.html#using-gpu-pass-through:~:text=All%20GPUs%20directly%20connected%20to%20each%20other%20through%20NVLink%20must%20be%20assigned%20to%20the%20same%20VM)

`All GPUs directly connected to each other through NVLink must be assigned to the same VM`

So, after removing NVlinks, its working as expected. Hope this helps some one.

 

[@mdrasheek](https://github.com/mdrasheek) We were running on a box with 8 GPUS and NVLinks. We are not doing VMs but running SLURM jobs that allocate cores/memory by cgroups and somehow isolate GPUs assigned too. My guess is that is similar to what the VMs do.

Still we don't want to remove the NVLinks as jobs that allocate mulitple GPUs should still be able to take advantage of them.

 

[@paulraines68](https://github.com/paulraines68) As you have mentioned you are using the driver version 530.x, which is not a datacenter driver. Check the supported products of driver 530.x in [this page](https://www.nvidia.com/download/driverResults.aspx/199985/en-us/) you will not find A100. You should use data center drivers on which they say the Xid119 errors are fixed. Refer [here](https://docs.nvidia.com/datacenter/tesla/drivers/index.html#cuda-drivers) to know more about datacenter drivers.

If you want to install using yum, you can refer to official installation instructions [here](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=CentOS&target_version=7&target_type=rpm_network).

 

I can confirm the issue on our machines running the 535 driver version. Disabling GSP workarounds the problem.

 

> I can confirm the issue on our machines running the 535 driver version. Disabling GSP workarounds the problem.

You can check which cards are interconnected with NVlinks using the command `nvidia-smi topo -m`. For example, GPU0 and GPU1 is connected with NV8, GPU2 and GPU3 is connected with NV12, if you allocate a job to GPU0&1, and another job to GPU2&3 it shouldn't have the issue. But when you mix up (like allocating jobs to GPU1 and GPU2) you might have an issue.

Hope this helps!

 

Just encountered this bug on A100 PCIE 80GB on all latest 525/530/535 official Linux drivers. Nvidia, this is a disgrace of a support for a 10K, or 13K (due to price gouging) gpu. Please fix this.

 

> Hi everyone, We have identified the issue related to our case with the help of Nvidia enterprise support team. In our machines, all the GPU cards are connected via NVLinks and as per the Nvidia doc [here](https://docs.nvidia.com/grid/13.0/grid-vgpu-user-guide/index.html#using-gpu-pass-through:~:text=All%20GPUs%20directly%20connected%20to%20each%20other%20through%20NVLink%20must%20be%20assigned%20to%20the%20same%20VM)
> 
> `All GPUs directly connected to each other through NVLink must be assigned to the same VM`
> 
> So, after removing NVlinks, its working as expected. Hope this helps some one.

Hi [@mdrasheek](https://github.com/mdrasheek) !  
I'm facing the same issue on A100: timeout from RPC and when I disable it, torch.cuda.is_available() takes 3 minutes to execute and other torch code is very slow on first initialization.  
Does removing NVlinks help to fix pytoch speed? And may be you've found some other workarounds?  
Thanks!

 

> > Hi everyone, We have identified the issue related to our case with the help of Nvidia enterprise support team. In our machines, all the GPU cards are connected via NVLinks and as per the Nvidia doc [here](https://docs.nvidia.com/grid/13.0/grid-vgpu-user-guide/index.html#using-gpu-pass-through:~:text=All%20GPUs%20directly%20connected%20to%20each%20other%20through%20NVLink%20must%20be%20assigned%20to%20the%20same%20VM)  
> > `All GPUs directly connected to each other through NVLink must be assigned to the same VM`  
> > So, after removing NVlinks, its working as expected. Hope this helps some one.
> 
> Hi [@mdrasheek](https://github.com/mdrasheek) ! I'm facing the same issue on A100: timeout from RPC and when I disable it, torch.cuda.is_available() takes 3 minutes to execute and other torch code is very slow on first initialization. Does removing NVlinks help to fix pytoch speed? And may be you've found some other workarounds? Thanks!

Yes, when we remove NVLinks we are able to use the cards in separate VMs.

Or check the NVLinks connectivity using `nvidia-smi topo -m`. If 2 cards interconnected using the same NVLinks, you can use those 2 cards in a single VM.

A sample screenshot is given below from 4x VM:

[![](https://private-user-images.githubusercontent.com/87429024/274874191-ef330dbb-47ce-4070-9cad-f9938be60c53.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MTE5NDE5MDksIm5iZiI6MTcxMTk0MTYwOSwicGF0aCI6Ii84NzQyOTAyNC8yNzQ4NzQxOTEtZWYzMzBkYmItNDdjZS00MDcwLTljYWQtZjk5MzhiZTYwYzUzLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDA0MDElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwNDAxVDAzMjAwOVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTE4MTYwZmI2NmIzNTg1MTViY2FlNGEwZTBjYTUwNmI0YmQzNThlYjY2MTBkMDM1ZWNiZGYwNDM0ZGEwMDg4MmYmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.Gc7APOo8sEhs5LKwtkVd4MJ8Cu_0f3La596aqaLi704)](https://private-user-images.githubusercontent.com/87429024/274874191-ef330dbb-47ce-4070-9cad-f9938be60c53.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MTE5NDE5MDksIm5iZiI6MTcxMTk0MTYwOSwicGF0aCI6Ii84NzQyOTAyNC8yNzQ4NzQxOTEtZWYzMzBkYmItNDdjZS00MDcwLTljYWQtZjk5MzhiZTYwYzUzLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDA0MDElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwNDAxVDAzMjAwOVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTE4MTYwZmI2NmIzNTg1MTViY2FlNGEwZTBjYTUwNmI0YmQzNThlYjY2MTBkMDM1ZWNiZGYwNDM0ZGEwMDg4MmYmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.Gc7APOo8sEhs5LKwtkVd4MJ8Cu_0f3La596aqaLi704)

You can see the GPU 0 and 1 is connected to an NVlink and GPU 2 and 3 are connected with another NVlink. So, the following deployment options will work

1.  VM launched with GPU 0 and 1
2.  VM launched with GPU 2 and 3
3.  VM with all the 4 cards.

Or as I mentioned earlier, if you are using single GPU for single process you can enable MIG, create MIG slices and use it acccordingly.

 

> Or check the NVLinks connectivity using nvidia-smi topo -m. If 2 cards interconnected using the same NVLinks, you can use those 2 cards in a single VM.

I rent VM with a single GPU from a cloud provider and `nvidia-smi topo -m` check shows that the connectivity for my GPU is 'Self' (X). Unfortunately, I'm still experiencing 3+ minutes delay on first `torch.cuda.is_available()` in a process.  
Although I can see in `nvidia-smi -i 0 -q` that `GPU Virtualization Mode` is `Pass-Through`. And the cloud works so, that my VM is not always assigned to some specific GPU, but it can vary on which one GPU from the A100s pool is free now.  
Not sure, but may be there is something behind the proccess of how it's organized on the cloud provider side, and I just can't directly see if there is NVLink.  
I also tested at other cloud provider VM and there were no problems with GSP timeout errors. The difference in terms of GPU on these VMs was: on a problematic VM there is A100-SXM4-80GB and `GPU Virtualization Mode` is `Pass-Through`, on the second one I had A100-PCIE-40GB and `GPU Virtualization Mode` - `None`

Haven't tried MIG slices yet, may be it will help.  
Thanks for your answer!

 

> > Or check the NVLinks connectivity using nvidia-smi topo -m. If 2 cards interconnected using the same NVLinks, you can use those 2 cards in a single VM.
> 
> I rent VM with a single GPU from a cloud provider and `nvidia-smi topo -m` check shows that the connectivity for my GPU is 'Self' (X). Unfortunately, I'm still experiencing 3+ minutes delay on first `torch.cuda.is_available()` in a process. Although I can see in `nvidia-smi -i 0 -q` that `GPU Virtualization Mode` is `Pass-Through`. And the cloud works so, that my VM is not always assigned to some specific GPU, but it can vary on which one GPU from the A100s pool is free now. Not sure, but may be there is something behind the proccess of how it's organized on the cloud provider side, and I just can't directly see if there is NVLink. I also tested at other cloud provider VM and there were no problems with GSP timeout errors. The difference in terms of GPU on these VMs was: on a problematic VM there is A100-SXM4-80GB and `GPU Virtualization Mode` is `Pass-Through`, on the second one I had A100-PCIE-40GB and `GPU Virtualization Mode` - `None`
> 
> Haven't tried MIG slices yet, may be it will help. Thanks for your answer!

SXM4 mostly comes in a bundle of 4 or 8 cards connected with NVlinks and it is possible that your cloud provider gives 1 card from it. You have to go with MIG and it should help you to overcome the lag issue.

 

for all the k8s users out there looking for gitOps way to fix this, were using the gpu-operator as a chart dependency in our own repo and added the following `configMap` to the templates deployed by our chart:

```
apiVersion: v1
kind: ConfigMap
data:
  nvidia.conf: |
    NVreg_EnableGpuFirmware=0
metadata:
  name: kernel-module-params

```

we also set the chart value `driver.kernelModuleConfig.name` to `kernel-module-params` in order to update the cluster-policy with the new `configMap` name.

it might require node reboot but it works perfectly.

 

> I want to highlight a point here, post enabling MIG and creating a full MIG Slice the GPU works without lag.
> 
> nvidia-smi -mig 1 nvidia-smi mig -cgi 0 -C
> 
> The above commands will create slice 0 which is the entire 80GB MIG slice. But with MIG mode disabled the lag comes again. I hope this may lead a way for the resolution for some one.

Thanks, this worked for me

 

I noticed the following, running in openstack with 4 A100 GPUs. I created 2 VMs, each with a GPU in there. I installed conda and pytorch for testing. When testing I open a python terminal and run the following code, do not close python leave it open.

```
import torch
torch.cuda.is_available()


```

Both driver 470, everything works, does not matter what order you do this.

Both driver 535, the first machine to run the python code has access to the GPU, the second machine fails.

Machine 1 has driver 470, Machine 2 has driver 535, if you start on machine 1 (470) and then machine 2 (535) the second machine can not get access to the GPU. If you first do machine 2 (535) and then machine 1 (470) both machines have access to the GPU.

Took a while to find the exact sequence, but hopefully this might help somebody to find out what the issue is. But it feels like driver 535 is locking all the GPUs, and driver 470 does not look at the lock, or just ignores it.

 

Nothing to preview