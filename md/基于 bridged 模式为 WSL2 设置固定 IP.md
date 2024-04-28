> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/670403474)

实测在 Windows11 23H2, WSL version 2.0.14.0, Ubuntu22.04 下有效

1.  添加 Hyper-V Virtual Switch

![](https://pic1.zhimg.com/v2-0d648ca61588e41c6c99cd6fea8e2060_r.jpg)

2. 通过`.wslconfig`设置 WSL2 网络模式为 bridged

```
[wsl2]
networkingMode=bridged
vmSwitch=WSLBridge # 改为第一步新建的交换机名称
dhcp=false
ipv6=true
macAddress=00:15:5d:13:71:52

```

3. 通过`/lib/systemd/network/wsl_external.network`为 WSL2 设置静态 IP

```
[Match]
Name=eth0
[Network]
Description=bridge
DHCP=false
Address=192.168.101.105/24 # 自行修改
Gateway=192.168.101.1 # 自行修改

```

4. 启用 systemd-networkd 配置网络，禁止 DNS 自动生成

```
# vim /etc/wsl.conf
[boot]
systemd=true # 使用 systemd
[network]
generateResolvConf=false # 禁止 DNS 自动生成
# 手动设置 DNS，随便选个可用的 DNS 服务器，比如 114.114.114.114
rm -rf /etc/resolv.conf
echo "nameserver 192.168.1.1" > /etc/resolv.conf
# 启用 systemd-networkd 配置网络
sudo systemctl enable systemd-networkd

```

5. `wsl --shutdown`重启 WSL2 即可。最终的网络拓扑如下：

![](https://pic1.zhimg.com/v2-0c06386e5fd1d0b5e24cd7d4a87215c4_r.jpg)

参考
--

[WSL With Bridging - Windows 11 - Microsoft Q&A](https://link.zhihu.com/?target=https%3A//learn.microsoft.com/en-us/answers/questions/788964/wsl-with-bridging-windows-11)

**[What is the Hyper-V Virtual Switch and How Does it Work?](https://link.zhihu.com/?target=https%3A//www.altaro.com/hyper-v/the-hyper-v-virtual-switch-explained-part-1/%23wait_approval)**

[Linux 虚拟网络设备 bridge 你真搞懂了吗？- 腾讯云开发者社区 - 腾讯云](https://link.zhihu.com/?target=https%3A//cloud.tencent.com/developer/article/1871867)

[WSL2 使用桥接网络，并指定 IP - 流年灬似氺 - 博客园](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/lic0914/p/17003251.html)