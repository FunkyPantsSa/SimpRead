> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/python2007cn/article/details/129528803)

[K3s](https://so.csdn.net/so/search?q=K3s&spm=1001.2101.3001.7020) server 配置

```
#vi /etc/rancher/k3s/registries.yaml
 
mirrors:
  "192.168.1.64:5000":
     endpoint:
      - "http://192.168.1.64:5000"
configs:
  "192.168.1.64:5000":
    auth:
      username: admin
      password: Harbor12345
```

systemctl restart k3s 服务

K3S agent 配置

k3s server 上查看 cat /var/lib/[rancher](https://so.csdn.net/so/search?q=rancher&spm=1001.2101.3001.7020)/k3s/agent/etc/containerd/config.toml

```
...
 
[plugins."io.containerd.grpc.v1.cri".registry.mirrors]
 
[plugins."io.containerd.grpc.v1.cri".registry.mirrors."192.168.1.64:5000"]
  endpoint = ["http://192.168.1.64:5000"]
 
[plugins."io.containerd.grpc.v1.cri".registry.configs."192.168.1.64:5000".auth]
  username = "admin"
  password = "Harbor12345"
```

复制最后三段内容到 k3s agent 机器上

vi /var/lib/rancher/k3s/agent/etc/containerd/config.toml

```
[plugins."io.containerd.grpc.v1.cri".registry.mirrors]
 
[plugins."io.containerd.grpc.v1.cri".registry.mirrors."192.168.1.64:5000"]
  endpoint = ["http://192.168.1.64:5000"]
 
[plugins."io.containerd.grpc.v1.cri".registry.configs."192.168.1.64:5000".auth]
  username = "admin"
  password = "Harbor12345"
```

systemctl restart k3s-agent

重启后生效