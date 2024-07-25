> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/easylife206/article/details/115774657)

公众号关注 「奇妙的 Linux 世界」  

设为「星标」，每天带你玩转 Linux ！

![](https://i-blog.csdnimg.cn/blog_migrate/8805746adc975d0b03e094fc155e172c.jpeg)  

> **在容器中使用 GPU 一直是使用 Compose 的一个痛点！**

  
在面向 `AI` 开发的大趋势下，容器化可以将环境无缝迁移，将配置环境的成本无限降低。但是，在容器中配置 `CUDA` 并运行 `TensorFlow` 一段时间内确实是个比较麻烦的时候，所以我们这里就介绍和使用它。

*   Enabling GPU access with Compose
    
*   Runtime options with Memory, CPUs, and GPUs
    
*   The Compose Specification
    
*   The Compose Specification - Deployment support
    
*   The Compose Specification - Build support
    

**在 Compose 中使用 GPU 资源**

*   如果我们部署 `Docker` 服务的的主机上正确安装并设置了其对应配置，且该主机上恰恰也有对应的 `GPU` 显卡，那么就可以在 `Compose` 中来定义和设置这些 `GPU` 显卡了。
    

```
# 需要安装的配置
$ apt-get install nvidia-container-runtime
```

*   **旧版本 <= 19.03**
    

```
# runtime
$ docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
```

*   **新版本 >= 19.03**
    

```
# with --gpus
$ docker run -it --rm --gpus all ubuntu nvidia-smi
 
# use device
$ docker run -it --rm --gpus \
    device=GPU-3a23c669-1f69-c64e-cf85-44e9b07e7a2a \
    ubuntu nvidia-smi
 
# specific gpu
$ docker run -it --rm --gpus '"device=0,2"' ubuntu nvidia-smi
 
# set nvidia capabilities
$ docker run --gpus 'all,capabilities=utility' --rm ubuntu nvidia-smi
```

*   对应 `Compose` 工具的老版本 (`v2.3`) 配置文件来说的话，想要在部署的服务当中使用 `GPU` 显卡资源的话，就必须使用 **`runtime`** 参数来进行配置才可以。虽然可以作为运行时为容器提供 `GPU` 的访问和使用，但是在该模式下并不允许对 `GPU` 设备的特定属性进行控制。
    

```
services:
  test:
    image: nvidia/cuda:10.2-base
    command: nvidia-smi
    runtime: nvidia
    environment:
      - NVIDIA_VISIBLE_DEVICES=all
```

> **在 `Compose v1.28.0+` 的版本中，使用 `Compose Specification` 的配置文件写法，并提供了一些可以更细粒度的控制 `GPU` 资源的配置属性可被使用，因此可以在启动的时候来精确表达我们的需求。咳咳咳，那这里我们就一起看看吧！**

*   `capabilities` - 必须字段
    
*   *   指定需要支持的功能；可以配置多个不同功能；必须配置的字段
        
    *   `man 7 capabilities`
        

```
deploy:
  resources:
    reservations:
      devices:
        - capabilities: ["gpu"]
```

*   `count`
    
*   *   指定需要使用的`GPU`数量；值为`int`类型；与`device_ids`字段二选一
        

```
deploy:
  resources:
    reservations:
      devices:
        - capabilities: ["tpu"]
          count: 2
```

*   `device_ids`
    
*   *   指定使用`GPU`设备`ID`值；与`count`字段二选一
        

```
deploy:
  resources:
    reservations:
      devices:
        - capabilities: ["gpu"]
          device_ids: ["0", "3"]
```

```
deploy:
  resources:
    reservations:
      devices:
        - capabilities: ["gpu"]
          device_ids: ["GPU-f123d1c9-26bb-df9b-1c23-4a731f61d8c7"]
```

*   `driver`
    
*   *   指定`GPU`设备驱动类型
        

```
deploy:
  resources:
    reservations:
      devices:
        - capabilities: ["nvidia-compute"]
          driver: nvidia
```

*   `options`
    
*   *   指定驱动程序的特定选项
        

```
deploy:
  resources:
    reservations:
      devices:
        - capabilities: ["gpu"]
          driver: gpuvendor
          options:
            virtualization: false
```

咳咳咳，看也看了，说也说了，那我们就简单的编写一个示例文件，让启动的 `cuda` 容器服务来使用一个 `GPU` 设备资源，并运行得到如下输出。

```
services:
  test:
    image: nvidia/cuda:10.2-base
    command: nvidia-smi
    deploy:
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
      resources:
        limits:
          cpus: "0.50"
          memory: 50M
        reservations:
          cpus: "0.25"
          memory: 20M
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu, utility]
      update_config:
        parallelism: 2
        delay: 10s
        order: stop-first
```

*   注意这里，如果设置 `count: 2` 的话，就会下面的输出中看到两块显卡设置的信息。如果，我们这里均未设置 `count` 或 `device_ids` 字段的话，则默认情况下将主机上所有 `GPU` 一同使用。
    

```
# 前台直接运行
$ docker-compose up
Creating network "gpu_default" with the default driver
Creating gpu_test_1 ... done
Attaching to gpu_test_1
test_1  | +-----------------------------------------------------------------------------+
test_1  | | NVIDIA-SMI 450.80.02    Driver Version: 450.80.02    CUDA Version: 11.1     |
test_1  | |-------------------------------+----------------------+----------------------+
test_1  | | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
test_1  | | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
test_1  | |                               |                      |               MIG M. |
test_1  | |===============================+======================+======================|
test_1  | |   0  Tesla T4            On   | 00000000:00:1E.0 Off |                    0 |
test_1  | | N/A   23C    P8     9W /  70W |      0MiB / 15109MiB |      0%      Default |
test_1  | |                               |                      |                  N/A |
test_1  | +-------------------------------+----------------------+----------------------+
test_1  |
test_1  | +-----------------------------------------------------------------------------+
test_1  | | Processes:                                                                  |
test_1  | |  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
test_1  | |        ID   ID                                                   Usage      |
test_1  | |=============================================================================|
test_1  | |  No running processes found                                                 |
test_1  | +-----------------------------------------------------------------------------+
gpu_test_1 exited with code 0
```

*   当然，如果设置了 `count` 或 `device_ids` 字段的话，就可以在容器里面的程序中使用多块显卡资源了。可以通过以下部署配置文件来进行验证和使用。
    

```
services:
  test:
    image: tensorflow/tensorflow:latest-gpu
    command: python -c "import tensorflow as tf;tf.test.gpu_device_name()"
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              device_ids: ["0", "3"]
              capabilities: [gpu]
```

*   运行结果，如下所示，我们可以看到两块显卡均可以被使用到。
    

```
# 前台直接运行
$ docker-compose up
...
Created TensorFlow device (/device:GPU:0 with 13970 MB memory -> physical GPU (device: 0, name: Tesla T4, pci bus id: 0000:00:1b.0, compute capability: 7.5)
...
Created TensorFlow device (/device:GPU:1 with 13970 MB memory) -> physical GPU (device: 1, name: Tesla T4, pci bus id: 0000:00:1e.0, compute capability: 7.5)
...
gpu_test_1 exited with code 0
```

> 本文转载自：「 Ecsape 的博客 」，原文：http://t.cn/A6c6d4l1 ，版权归原作者所有。欢迎投稿，投稿邮箱: editor@hi-linux.com。

  
![](https://i-blog.csdnimg.cn/blog_migrate/94c242c2d176b89f0acb280c56347597.gif)  

![](https://i-blog.csdnimg.cn/blog_migrate/137008597fd31d706c5fa949f892e5ee.png)

**你可能还喜欢**

点击下方图片即可阅读  

[![](https://i-blog.csdnimg.cn/blog_migrate/3b22d869961b89eb94b79d461524970c.png)](http://mp.weixin.qq.com/s?__biz=MzI3MTI2NzkxMA%3D%3D&chksm=eac6cceeddb145f817f5257296b298ba154962a739a3feb1bb08061e1320192543bba718d0d8&idx=1&mid=2247495111&scene=21&sn=40ef75da0e8f1fa5a85be190c46e1867#wechat_redirect)

微软推出 Go 语言免费中文教程，真香！  

![](https://i-blog.csdnimg.cn/blog_migrate/b03d63a7ab763a1d5b80133a99010d61.jpeg)

点击上方图片，打开小程序，加入「玩转 Linux」圈子  

![](https://i-blog.csdnimg.cn/blog_migrate/3fad7e10a2b101fade2b2a0b8d0a1731.png)

更多有趣的互联网新鲜事，关注「奇妙的互联网」视频号全了解！