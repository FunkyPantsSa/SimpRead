> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.hiascend.com](https://www.hiascend.com/document/detail/zh/quick-installation/24.0.RC1/quickinstg/500_Pro_3000/quickinstg_500_Pro_3000_0007.html)

> 安装 NPU 驱动固件 安装说明 首次安装场景：硬件设备刚出厂时未安装驱动，或者硬件设备前期安装过驱动固件但是当前已卸载，上述场景属于首次安装场景，需按照 “驱动> 固件” 的顺序安装驱动固件。

#### 安装说明

*   首次安装场景：硬件设备刚出厂时未安装驱动，或者硬件设备前期安装过驱动固件但是当前已卸载，上述场景属于首次安装场景，需按照 **“驱动> 固件”** 的顺序安装驱动固件。
*   覆盖安装场景：硬件设备前期安装过驱动固件且未卸载，当前要再次安装驱动固件，此场景属于覆盖安装场景，需按照 **“固件** **> 驱动”** 的顺序安装固件驱动。

#### 检查安装环境

在安装驱动固件前，建议按照以下检查项检查环境，确保驱动固件能正常安装。

<table><caption><b>表 1 </b>环境检查</caption><thead><tr><th><p>检查项</p></th><th><p>检查方法</p></th></tr></thead><tbody><tr><td headers="mcps1.3.2.3.2.3.1.1 "><p>检测卡是否正常在位</p></td><td headers="mcps1.3.2.3.2.3.1.2 "><p>可通过命令行或 BMC 管理页面检测卡是否正常在位，如果执行命令时，提示没有安装<strong> lspci</strong>，可通过 BMC 管理页面检查。</p><ul><li>执行<strong> lspci | grep d500</strong> 命令，如果服务器上有<em> N</em>（<em>N</em>＞0）张卡，回显中含 “d500” 字段的行数为<em> N</em>，则表示卡正常在位。<pre>01:00.0 Processing accelerators: Huawei Technologies Co., Ltd. Device d500 (rev 23)
......
</pre></li><li>如果卡所在的服务器是华为服务器，可通过 BMC 管理网口登录 iBMC WebUI 界面，选择 “系统管理 &gt; 系统信息”，单击 “其他”。若推理卡的 PCIe 卡信息在 “PCIe 卡” 列表中，表示卡正常在位。</li></ul></td></tr><tr><td headers="mcps1.3.2.3.2.3.1.1 "><p>操作系统相关配置文件（重要）</p></td><td headers="mcps1.3.2.3.2.3.1.2 "><p>如果环境 OS 为<strong> Linx 6.0.90</strong><strong>/</strong><strong>Linx 6.0.100</strong> 操作系统，需要检查 “/etc/pam.d/su” 配置文件的 “auth sufficient pam_rootok.so” 字段是否被注释掉，若被注释，将 “auth sufficient pam_rootok.so” 前面 "<strong>#</strong>" 号删除，保存退出。</p></td></tr></tbody></table>

#### 确认操作系统和内核版本

在安装驱动前，需要用户确认现场操作系统和内核版本，从而确定是否需要安装驱动编译所需依赖。

执行如下命令查看现场服务器操作系统和内核版本，并和华为的版本要求进行对比。

```
uname -m && cat /etc/*release
uname -r

```

Atlas 500 Pro 智能边缘服务器（型号：3000）OS 兼容性如[表 2](#ZH-CN_TOPIC_0000001861874257__zh-cn_topic_0000001265700710_table164904417616) 所示。

操作系统内核版本和对应的安装方式如[表 3](#ZH-CN_TOPIC_0000001861874257__zh-cn_topic_0000001265700710_zh-cn_topic_0000001261590847_table137358168346) 所示（推理卡 OS 兼容性范围不止表中呈现的范围，表中只保留 Atlas 500 Pro 支持的 OS）。

![](https://www.hiascend.com/doc_center/source/zh/quick-installation/24.0.RC1/quickinstg/500_Pro_3000/public_sys-resources/notice_3.0-zh-cn.png)

如果现场服务器连通外网，Ubuntu 系统会自动升级内核，因此安装 NPU 驱动前，需要检查内核版本，如果内核版本和配套表中的不一致，需要使用源码编译方式安装。同时建议关闭内核自动更新机制，避免更新后 NPU 驱动不可用。

执行如下命令关闭内核自动更新：

```
apt-mark hold linux-image-generic linux-headers-generic linux-image-extra

```

<table><caption><b>表 2 </b>Atlas 500 Pro 智能边缘服务器（型号：3000）</caption><thead><tr><th><p>host 操作系统版本</p></th><th><p>host 操作系统架构</p></th><th><p>软件包默认的 host 操作系统内核版本</p></th><th><p>gcc 编译器版本</p></th></tr></thead><tbody><tr><td headers="mcps1.3.3.7.2.5.1.1 "><p>CentOS 7.6</p></td><td headers="mcps1.3.3.7.2.5.1.2 "><p>aarch64</p></td><td headers="mcps1.3.3.7.2.5.1.3 "><p>4.14.0-115.el7a.0.1.aarch64</p></td><td headers="mcps1.3.3.7.2.5.1.4 "><p>4.8.5</p></td></tr><tr><td headers="mcps1.3.3.7.2.5.1.1 "><p>Ubuntu 20.04</p></td><td headers="mcps1.3.3.7.2.5.1.2 "><p>aarch64</p></td><td headers="mcps1.3.3.7.2.5.1.3 "><p>5.4.0-26-generic</p></td><td headers="mcps1.3.3.7.2.5.1.4 "><p>原生 gcc（源自带 gcc 版本）</p></td></tr><tr><td headers="mcps1.3.3.7.2.5.1.1 "><p>openEuler 20.03 LTS</p></td><td headers="mcps1.3.3.7.2.5.1.2 "><p>aarch64</p></td><td headers="mcps1.3.3.7.2.5.1.3 "><p>4.19.90-2003.4.0.0036.oe1.aarch64</p></td><td headers="mcps1.3.3.7.2.5.1.4 "><p>7.3.0</p></td></tr><tr><td headers="mcps1.3.3.7.2.5.1.1 "><p>UOS 20</p></td><td headers="mcps1.3.3.7.2.5.1.2 "><p>aarch64</p></td><td headers="mcps1.3.3.7.2.5.1.3 "><p>4.19.0-arm64-server</p></td><td headers="mcps1.3.3.7.2.5.1.4 "><p>8.3.0</p></td></tr><tr><td headers="mcps1.3.3.7.2.5.1.1 "><p>Kylin V10 SP1</p></td><td headers="mcps1.3.3.7.2.5.1.2 "><p>aarch64</p></td><td headers="mcps1.3.3.7.2.5.1.3 "><p>4.19.90-17.ky10.aarch64</p></td><td headers="mcps1.3.3.7.2.5.1.4 "><p>7.3.0</p></td></tr><tr><td headers="mcps1.3.3.7.2.5.1.1 "><p>CentOS 8.2</p></td><td headers="mcps1.3.3.7.2.5.1.2 "><p>aarch64</p></td><td headers="mcps1.3.3.7.2.5.1.3 "><p>4.18.0-193.el8.aarch64</p></td><td headers="mcps1.3.3.7.2.5.1.4 "><p>8.3.1</p></td></tr><tr><td headers="mcps1.3.3.7.2.5.1.1 "><p>Linx 6.0.90</p></td><td headers="mcps1.3.3.7.2.5.1.2 "><p>aarch64</p></td><td headers="mcps1.3.3.7.2.5.1.3 "><p>4.19.0-0.bpo.1-linx-security-arm64</p></td><td headers="mcps1.3.3.7.2.5.1.4 "><p>6.3.0</p></td></tr><tr><td headers="mcps1.3.3.7.2.5.1.1 "><p>Linx 6.0.100</p></td><td headers="mcps1.3.3.7.2.5.1.2 "><p>aarch64</p></td><td headers="mcps1.3.3.7.2.5.1.3 "><p>4.19.0-11-linx-security-arm64</p></td><td headers="mcps1.3.3.7.2.5.1.4 "><p>8.3.0</p></td></tr></tbody></table><table><caption><b>表 3 </b>Atlas 300I Pro 推理卡 /Atlas 300V Pro 视频解析卡</caption><thead><tr><th><p>host 操作系统版本</p></th><th><p>host 操作系统架构</p></th><th><p>软件包默认的 host 操作系统内核版本</p></th><th><p>gcc 编译器版本</p></th><th><p>安装方式</p></th></tr></thead><tbody><tr><td headers="mcps1.3.3.8.2.6.1.1 "><p>CentOS 7.6</p></td><td headers="mcps1.3.3.8.2.6.1.2 "><p>aarch64</p></td><td headers="mcps1.3.3.8.2.6.1.3 "><p>4.14.0-115.el7a.0.1.aarch64</p></td><td headers="mcps1.3.3.8.2.6.1.4 "><p>4.8.5</p></td><td rowspan="2" headers="mcps1.3.3.8.2.6.1.5 "><p>二进制安装。</p><p>直接按照<a href="#ZH-CN_TOPIC_0000001861874257__zh-cn_topic_0000001472911169_section759123593114" data-outer="inner" data-href="#ZH-CN_TOPIC_0000001861874257__zh-cn_topic_0000001472911169_section759123593114" data-anchor="true">安装驱动固件</a>内容安装驱动固件。</p></td></tr><tr><td headers="mcps1.3.3.8.2.6.1.1 "><p>Ubuntu 20.04</p></td><td headers="mcps1.3.3.8.2.6.1.2 "><p>aarch64</p></td><td headers="mcps1.3.3.8.2.6.1.3 "><p>5.4.0-26-generic</p></td><td headers="mcps1.3.3.8.2.6.1.4 "><p>原生 gcc（源自带 gcc 版本）</p></td></tr><tr><td headers="mcps1.3.3.8.2.6.1.1 "><p>openEuler 20.03 LTS</p></td><td headers="mcps1.3.3.8.2.6.1.2 "><p>aarch64</p></td><td headers="mcps1.3.3.8.2.6.1.3 "><p>4.19.90-2003.4.0.0036.oe1.aarch64</p></td><td headers="mcps1.3.3.8.2.6.1.4 "><p>7.3.0</p></td><td rowspan="4" headers="mcps1.3.3.8.2.6.1.5 "><p>源码编译安装。</p><ol><li>需要先参见<a href="https://www.hiascend.com/document/detail/zh/quick-installation/24.0.RC1/quickinstg/500_Pro_3000/quickinstg_500_Pro_3000_0051.html" data-outer="inner" data-href="quickinstg_500_Pro_3000_0051.html" data-doc="true" target="_blank" data-multiple-screen="true">安装驱动源码编译所需依赖</a>安装 dkms 等依赖。</li><li>再按照<a href="#ZH-CN_TOPIC_0000001861874257__zh-cn_topic_0000001472911169_section759123593114" data-outer="inner" data-href="#ZH-CN_TOPIC_0000001861874257__zh-cn_topic_0000001472911169_section759123593114" data-anchor="true">安装驱动固件</a>内容安装驱动固件。</li></ol></td></tr><tr><td headers="mcps1.3.3.8.2.6.1.1 "><p>Kylin V10 SP1</p></td><td headers="mcps1.3.3.8.2.6.1.2 "><p>aarch64</p></td><td headers="mcps1.3.3.8.2.6.1.3 "><p>4.19.90-17.ky10.aarch64</p></td><td headers="mcps1.3.3.8.2.6.1.4 "><p>7.3.0</p></td></tr><tr><td headers="mcps1.3.3.8.2.6.1.1 "><p>Linx 6.0.90</p></td><td headers="mcps1.3.3.8.2.6.1.2 "><p>aarch64</p></td><td headers="mcps1.3.3.8.2.6.1.3 "><p>4.19.0-0.bpo.1-linx-security-arm64</p></td><td headers="mcps1.3.3.8.2.6.1.4 "><p>6.3.0</p></td></tr><tr><td headers="mcps1.3.3.8.2.6.1.1 "><p>Linx 6.0.100</p></td><td headers="mcps1.3.3.8.2.6.1.2 "><p>aarch64</p></td><td headers="mcps1.3.3.8.2.6.1.3 "><p>4.19.0-11-linx-security-arm64</p></td><td headers="mcps1.3.3.8.2.6.1.4 "><p>8.3.0</p></td></tr></tbody></table>

#### 安装驱动固件

1.  以 **root** 用户登录服务器。
2.  创建驱动运行用户 HwHiAiUser（运行驱动进程的用户），安装驱动时无需指定运行用户，默认即为 HwHiAiUser。
    
    ```
    groupadd HwHiAiUser
    useradd -g HwHiAiUser -d /home/HwHiAiUser -m HwHiAiUser -s /bin/bash
    
    ```
    
3.  将驱动包和固件包上传到服务器任意目录如 “/home”。
4.  进入驱动包和固件包所在目录，执行如下命令，增加驱动和固件包的可执行权限。
    
    ```
    chmod +x Ascend-hdk-310p-npu-driver_24.1.rc1_linux-aarch64.run
    chmod +x Ascend-hdk-310p-npu-firmware_7.1.0.6.220.run
    
    ```
    
5.  执行以下命令，完成驱动固件安装，软件包默认安装路径为 “/usr/local/Ascend”。
    *   安装驱动
        
        执行以下命令，完成驱动安装。
        
        ```
        ./Ascend-hdk-310p-npu-driver_24.1.rc1_linux-aarch64.run --full  --install-for-all
        
        ```
        
        *   若执行上述安装命令出现类似如下回显信息，请参见[驱动安装缺少依赖报错](https://www.hiascend.com/document/detail/zh/quick-installation/24.0.RC1/quickinstg/500_Pro_3000/quickinstg_500_Pro_3000_0057.html)解决。
            
            ```
            [ERROR]The list of missing tools: lspci,ifconfig,
            
            ```
            
        *   若执行上述安装命令出现类似如下回显信息，请参见[驱动安装过程中出现 dkms 编译失败报错](https://www.hiascend.com/document/detail/zh/quick-installation/24.0.RC1/quickinstg/500_Pro_3000/quickinstg_500_Pro_3000_0058.html)解决。
            
            ```
            [ERROR]Dkms install failed, details in : var/log/ascend_seclog/ascend_install.log. 
            [ERROR]Driver_ko_install failed, details in : /var/log/ascend_seclog/ascend_install.log.
            
            ```
            
        *   若系统出现如下关键回显信息，则表示驱动安装成功。
            
            ```
            Driver package installed successfully!
            
            ```
            
    *   安装固件
        
        执行以下命令，完成固件安装。
        
        ```
        ./Ascend-hdk-310p-npu-firmware_7.1.0.6.220.run --full
        
        ```
        
        若系统出现如下关键回显信息，表示固件安装成功。
        
        ```
        Firmware package installed successfully! Reboot now or after driver installation for the installation/upgrade to take effect 
        
        ```
        
6.  执行 **reboot** 命令重启系统。
7.  执行 **npu-smi info** 查看驱动加载是否成功。
    
    若出现类似如下图所示回显信息，说明加载成功。否则，说明加载失败。请联系华为技术支持处理。
    
    ![](https://www.hiascend.com/doc_center/source/zh/quick-installation/24.0.RC1/quickinstg/500_Pro_3000/figure/zh-cn_image_0000001815074450.jpg)
    

#### 升级 MCU

MCU 是推理卡带外管理模块，具备单板监测、故障上报等功能。出厂时推理卡已集成了初始版本，为了保障所有功能正常使用，请将 MCU 升级到配套版本。

本章内容主要介绍通过 npu-smi 工具升级 MCU，npu-smi 工具可以将单个推理卡的 MCU 升级到相应版本，如果配备了多个推理卡，需要逐个升级。

1.  将获取的 zip 包解压至本地文件夹，获取安装包。
    
    _Ascend-hdk-310p-mcu_23.2.4.hpm_
    
2.  以 **root** 用户登录服务器，将安装包上传至 Linux 系统任意目录下（如 “/home”）。
3.  执行 **npu-smi info -l** 命令查询 _NPU ID_（推理卡的设备编号）。
    
    回显类似如下信息，_NPU ID_ 为 **8**。
    
    ```
            Card Count                     : 1
            NPU ID : 8
            Product Name                   : IT21DMPB01
            Serial Number                  : 033EFS10M8000087
            Chip Count                     : 4
    
    ```
    
4.  进入 MCU 软件包所在路径，执行如下命令启动升级（将 _NPU ID_ 替换为 [3](#ZH-CN_TOPIC_0000001861874257__zh-cn_topic_0000001815073886_li7169181141512) 中查询到的设备编号）。
    
    ```
    npu-smi upgrade -t mcu -i NPU ID -f Ascend-hdk-310p-mcu_23.2.4.hpm
    
    ```
    
    出现类似如下回显表示升级成功。
    
    ```
    Start upgrade [100].
            Status                         : OK
            Message                        : The device upgrade is started successfully
            Message                        : need active mcu
    
    ```
    
5.  执行如下命令使新版本生效，类似以下回显表示已生效。
    
    **npu-smi upgrade -a mcu -i** _NPU ID_
    
    ```
    Status                         : OK
            Message                : The upgrade has taken effect after performed reboot successfully.
    
    ```
    
6.  在生效新版本之后，等待 30s，查询 MCU 版本号，确保升级成功。
    
    **npu-smi upgrade -b mcu -i** _NPU ID_
    
    ```
    Version                        : 23.2.4
    
    ```
    
    ![](https://www.hiascend.com/doc_center/source/zh/quick-installation/24.0.RC1/quickinstg/500_Pro_3000/public_sys-resources/note_3.0-zh-cn.png)
    
    *   MCU 新版本生效后，如需再次升级，请等待 5min 后再次操作。
    *   如果升级后不是目标版本或者升级失败，请重新进行升级。如果依然升级失败，请记录故障现象和操作步骤，并联系华为技术支持解决。