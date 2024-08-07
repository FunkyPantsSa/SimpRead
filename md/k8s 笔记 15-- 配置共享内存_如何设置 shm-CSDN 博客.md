> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/u011127242/article/details/124367275)

#### [k8s](https://so.csdn.net/so/search?q=k8s&spm=1001.2101.3001.7020) 笔记 15-- 配置共享内存

*   [介绍](#_1)
*   [案例](#_5)
*   *   [docker 环境](#docker__6)
    *   [k8s 环境](#k8s__14)
*   [说明](#_44)

介绍
--

容器启动后默认会有 64M 的[共享内存](https://so.csdn.net/so/search?q=%E5%85%B1%E4%BA%AB%E5%86%85%E5%AD%98&spm=1001.2101.3001.7020)挂载在 / dev/shm 目录，用户可以向访问本地文件系统一样访问该共享内存，如果需要配置更大的内存，那么在 docker 中需要通过 shm-size 来获取，k8s 中需要通过挂载 memory 类型的 emptyDir 来实现。  
本文主要介绍如何配置 docker 和 k8s 环境的共享内存目录，并加以案例说明。

案例
--

### docker 环境

默认为 64M，如下图：  
![](https://i-blog.csdnimg.cn/blog_migrate/d5f3e73f749ceef63029f2500ada6124.png)  
通 shm-size 配置共享内存大小：

```
docker run -d --name=test --shm-size=500m busybox:1.32 sleep infinity

```

![](https://i-blog.csdnimg.cn/blog_migrate/ed2eb394f82f2650e646d261bcc3c065.png)

### k8s 环境

配置 emptyDir， medium 为 Memory

```
apiVersion: v1
kind: Pod
metadata:
  name: test-shm
spec:
  containers:
  - image: busybox:1.32
    name: test-container
    command: [sh, -c, "sleep infinity" ]
    volumeMounts:
    - mountPath: /dev/shm
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      medium: Memory
      sizeLimit: 500M

```

如下图，容器中正常申请到了 500M 的共享内存。  
![](https://i-blog.csdnimg.cn/blog_migrate/4fcefa4ab94d599ff0789f99306f94d9.png)  
注意：

1.  k8s 中 M 和 Mi 的单位是不同的，加 i 后是 1024 的基数，不加则为 1000 的基数。  
    若设置 sizeLimit: 500Mi , 那么大小为 500M， 如下图：  
    ![](https://i-blog.csdnimg.cn/blog_migrate/6b3a47eac33df855d9642e68c893dcd6.png)
2.  k8s 1.22 版本中才比较完整的支持挂载共享内存，即 1.22 及之后的的版本挂载的大小为实际 sizeLimit 的大小， 1.20 版本中会出现挂载后实际大小和物理机一样 (物理内存一半) 的现象。

说明
--

[docs.docker.com/engine/reference/commandline/run/](https://docs.docker.com/engine/reference/commandline/run/)  
[kubernetes.io/docs/concepts/storage/volumes#emptydir](https://kubernetes.io/docs/concepts/storage/volumes#emptydir)