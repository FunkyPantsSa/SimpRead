> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/MssGuo/article/details/131026874)

#### 目录

*   *   [前言](#_1)
    *   [docker 与 containerd 的命令区别](#docker_containerd_4)
    *   [安装 containerd](#containerd_14)
    *   [容器 CLI](#CLI_44)
    *   [ctr 全局命令概览](#ctr_50)
    *   [containerd 常用命令](#containerd_86)
    *   [ctr namespaces 管理命名空间](#ctr_namespaces__89)
    *   [ctr images 镜像管理命令](#ctr_images__110)
    *   *   [ctr images ls 列出镜像](#ctr_images_ls__130)
        *   [ctr images pull 下载镜像](#ctr_images_pull__136)
        *   [ctr images mount 挂载镜像](#ctr_images_mount__169)
        *   [ctr images unmount 卸载镜像](#ctr_images_unmount__175)
        *   [ctr images export 导出镜像](#ctr_images_export__181)
        *   [ctr images import 导入镜像](#ctr_images_import__188)
        *   [ctr images rm 删除镜像](#ctr_images_rm__194)
        *   [ctr images tag 修改镜像 tag](#ctr_images_tag_tag_202)
        *   [ctr images check 检查镜像](#ctr_images_check__210)
*   [ctr containers 容器管理命令](#ctr_containers__216)
*   *   *   [ctr containers create 创建容器](#ctr_containers_create__236)
        *   [ctr containers ls 列出容器](#ctr_containers_ls__285)
        *   [ctr containers info 获取容器信息](#ctr_containers_info__294)
        *   [ctr containers rm 删除容器](#ctr_containers_rm__299)
    *   [ctr tasks 管理任务](#ctr_tasks__311)
    *   *   [ctr tasks start 启动已经创建的容器](#ctr_tasks_start__336)
        *   [ctr tasks list, ls 查看任务](#ctr_tasks_list_ls__354)
        *   [ctr tasks ps 查看 / 列出容器进程](#ctr_tasks_ps___374)
        *   [ctr tasks pause 暂停容器运行](#ctr_tasks_pause___390)
        *   [ctr tasks resume 恢复暂停的容器](#ctr_tasks_resume____404)
        *   [ctr tasks kill 发送信号给容器 (default: SIGTERM)](#ctr_tasks_kill____default_SIGTERM_419)
        *   [ctr tasks delete, del, remove, rm 删除任务](#ctr_tasks_delete_del_remove_rm___440)
        *   [ctr tasks exec 进入容器](#ctr_tasks_exec___461)
    *   [ctr run 直接运行容器](#ctr_run__483)

### 前言

k8s 从 1.24 版本开始默认是 containerd 作为容器运行时，不在使用 docker 作为容器运行时。所以，本篇讲解 containerd 的基本安装及使用方法。

### docker 与 containerd 的命令区别

```
1、docker默认是docker.io官网去找镜像，所以docker拉去镜像可以这样写：docker pull nginx:1.18，而containerd没有默认仓库，所以必须写
完整的镜像名字才能拉去镜像，如：ctr images pull  docker.io/library/nginx:1.18。
2、docker run一个容器时，如果没有镜像则会去拉去镜像，而containerd不会。
3、containerd有命名空间的概念，镜像和容器不要在同一个命名空间。
4、containerd支持mount镜像到挂载点，方便查看镜像里面的文件内容（这点个人感觉很不错）
5、ctr images export、ctr images import 是导出镜像和导入镜像，其实等价于docker save、docker load.

```

### 安装 containerd

containerd 没有单独的 yum 源，我们可以从 docker-ce 的 yum 源中安装 containerd。

```
#安装docker-ce的阿里云yum源
wget -O /etc/yum.repos.d/docker-ce.repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#查看yum源中containerd
yum list | grep containerd
#yum安装containerd（安装containerd.io即是安装containerd）
yum install containerd.io
#启动containerd并设置开机自启
systemctl start containerd
systemctl enable containerd

#查看containerd安装了哪些文件
[root@localhost ~]# rpm -qa | grep contain
containerd.io-1.6.21-3.1.el7.x86_64
container-selinux-2.119.2-1.911c772.el7_8.noarch
[root@localhost ~]# rpm -ql containerd.io-1.6.21-3.1.el7.x86_64
/etc/containerd											##containerd的配置文件目录
/etc/containerd/config.toml								#containerd的配置文件
/usr/bin/containerd										#containerd的可自行文件
/usr/bin/containerd-shim
/usr/bin/containerd-shim-runc-v1
/usr/bin/containerd-shim-runc-v2
/usr/bin/ctr											#containerd的客户端工具
/usr/bin/runc
/usr/lib/systemd/system/containerd.service				#服务启动文件
[root@localhost ~]# 

```

### 容器 CLI

CLI 是`"Command Line Interface"`的英文缩写，意思是 "命令行界面"，常见的容器 CLI 包含以下几种：  
docker 使用 [docker 命令](https://so.csdn.net/so/search?q=docker%E5%91%BD%E4%BB%A4&spm=1001.2101.3001.7020)管理容器和镜像；  
containerd 使用 [ctr](https://so.csdn.net/so/search?q=ctr&spm=1001.2101.3001.7020) 命令管理容器和镜像，CLI 命令行工具；  
k8s 中使用 crictl 命令管理容器和镜像；  
ctr 是 containerd 自带的 CLI 命令行工具，crictl 是 k8s 中 CRI（容器运行时接口）的客户端，k8s 使用该客户端和 containerd 进行交互。

### ctr 全局命令概览

安装 containerd 默认安装了 ctr 命令，ctr 命令是 containerd 的客户端管理工具，主要用于管理容器及镜像等。

```
#查看containerd客户端即服务器版本信息
ctr version
#ctr全局命令概览
[root@localhost ~]# ctr --help
USAGE:	(注意看这个格式：ctr 全局选项 命令 命令选项 参数)
   ctr [global options] command [command options] [arguments...]
COMMANDS:
   plugins, plugin            provides information about containerd plugins
   version                    打印客户端和服务端版本
   containers, c, container   管理containers
   content                    管理content
   events, event              显示containerd events
   images, image, i           管理镜像
   leases                     管理leases
   namespaces, namespace, ns  管理namespaces
   pprof                      provide golang pprof outputs for containerd
   run                        运行一个container
   snapshots, snapshot        管理snapshots
   tasks, t, task             管理tasks
   install                    安装一个新的package
   oci                        OCI tools
   shim                       interact with a shim directly
   help, h                    显示命令帮助

全局选项:
   --debug                      在logs中开启enable输出
   --address value, -a value    containerd's GRPC server的地址 (默认: "/run/containerd/containerd.sock") 
   --timeout value              ctr命令超时时间(默认: 0s)
   --connect-timeout value      连接containerd超时时间 (默认: 0s)
   --namespace value, -n value  命名空间 (默认: "default")
   --help, -h                   显示帮助
   --version, -v                打印版本信息

```

### containerd 常用命令

由于 containerd 有命名空间的概念，这一点与 docker 不同，所以前面先来讲命名空间。

### ctr namespaces 管理命名空间

```
#namespaces, namespace, ns 都可以
[root@localhost ~]# ctr namespaces --help
NAME:
   ctr namespaces - manage namespaces
USAGE:
   ctr namespaces command [command options] [arguments...]
COMMANDS:
   create, c   创建新的命名空间（命名空间名字必须是唯一的）
   list, ls    列出命名空间
   remove, rm  删除一个或多个命名空间（命名空间必须为空）
   label       设置和删除命名空间的labels（labels是key=value形式，key=空value表示删除）

#演示示例   
ctr namespace ls
ctr namespace create helm 
ctr namespace remove helm 
ctr namespace label  helm k8s=true  	#ctr namespaces label <name> [<key>=<value>, ...]
ctr namespace label  helm k8s=  		#k8s=空值,表示删除该key

```

### ctr images 镜像管理命令

containerd 支持 oci 标准的镜像，所以可以直接使用 docker 官方或 Dockerfile 构建的镜像。

```
[root@localhost ~]# ctr images
USAGE:
   ctr images command [command options] [arguments...]
COMMANDS:
   check                    check existing images to ensure all content is available locally
   export                   export images
   import                   import images
   list, ls                 列出镜像
   mount                    挂载一个镜像到指定路径（和挂载分区一个道理）
   unmount                  卸载镜像
   pull                     拉取镜像
   push                     推送镜像
   delete, del, remove, rm  删除镜像
   tag                      给镜像打tag
   label                    设置和清楚镜像的labels
   convert                  convert an image

```

#### ctr images ls 列出镜像

```
#ctr images ls 列出镜像（不指定命名空间默认就是default）
ctr -n helm images list		
ctr images ls

```

#### ctr images pull 下载镜像

```
#ctr images pull下载镜像
#镜像名称需要写完整路径，docker中即使不写完整路径也是默认从docker.io中下载，containerd必须要写完整镜像名称
root@localhost ~]# ctr -n helm images pull --help	#查看下载镜像帮助
NAME:
   ctr images pull - pull an image from a remote
USAGE:
   ctr images pull [command options] [flags] <ref>
OPTIONS:
   --skip-verify, -k                 skip SSL certificate validation
   --plain-http                      allow connections using plain HTTP
   --user value, -u value            镜像仓库的用户密码，格式 user[:password]
   --refresh value                   refresh token for authorization server
   --hosts-dir value                 Custom hosts configuration directory
   --tlscacert value                 path to TLS root CA
   --tlscert value                   path to TLS client certificate
   --tlskey value                    path to TLS client key
   --http-dump                       dump all HTTP request/responses when interacting with container registry
   --http-trace                      enable HTTP tracing for registry interactions
   --snapshotter value               快照的名字。空值表示默认值。[$CONTAINERD_SNAPSHOTTER]
   --label value                     附加到镜像的labels 
   --platform value                  从指定平台拉取镜像内容
   --all-platforms                   从全部平台拉取镜像内容和元数据，不写默认根据当前系统架构而定
   --all-metadata                    从全部平台拉取镜像元数据
   --print-chainid                   Print the resulting image's chain ID
   --max-concurrent-downloads value  设置每次拉取的最大并发下载(默认:0)

#示例
ctr images pull docker.io/library/nginx:1.18			#默认下载到default命名空间
ctr -n helm images pull docker.io/library/nginx:1.18	#默认下载到default命名空间
ctr images pull --platform linux/amd64 docker.io/library/nginx:1.18	#下载指定平台，我们一般不会指定平台，让其默认平台即可

```

#### ctr images mount 挂载镜像

```
# ctr images mount 挂载镜像
#containerd可以将一个镜像挂载到某个挂载点，用于方便查看该镜像有什么内容
ctr images mount  docker.io/library/nginx:1.18 /mnt/

```

#### ctr images unmount 卸载镜像

```
#卸载镜像
ctr images unmount  /mnt/
umount /mnt

```

#### ctr images export 导出镜像

```
#导出镜像
ctr images export nginx:1.18.tar.gz  docker.io/library/nginx:1.18
ctr images export --platform linux/amd64 nginx:1.18.tar.gz  docker.io/library/nginx:1.18
ctr images export --all-platforms  nginx:1.18.tar.gz  docker.io/library/nginx:1.18

```

#### ctr images import 导入镜像

```
#导入镜像(有疑问，导入报错)
ctr images import  nginx:1.18.tar.gz
ctr -n helm images import  nginx:1.18.tar.gz

```

#### ctr images rm 删除镜像

```
#ctr images rm 删除镜像
ctr images delete docker.io/library/nginx:1.18
ctr images del docker.io/library/nginx:1.18
ctr images remove  docker.io/library/nginx:1.18
ctr images rm docker.io/library/nginx:1.18

```

#### ctr images tag 修改镜像 tag

```
#ctr images tag 修改镜像tag
语法：
ctr images tag [command options] [flags] <source_ref> <target_ref> [<target_ref>, ...]
示例：
ctr images tag docker.io/library/nginx:1.18  192.168.110.110/nginx/nginx:1.18

```

#### ctr images check 检查镜像

```
#ctr images check 检查镜像
#检查镜像确保所有内容在本地可用
ctr images check

```

ctr containers 容器管理命令
---------------------

```
containers, c, container
[root@localhost ~]# ctr containers --help
NAME:
   ctr containers - manage containers
USAGE:
   ctr containers command [command options] [arguments...]
COMMANDS:
   create                   创建容器
   delete, del, remove, rm  删除一个或多个容器
   info                     获取容器信息
   list, ls                 列出容器
   label                    设置或删除容器labels
   checkpoint               checkpoint a container
   restore                  restore a container from checkpoint
OPTIONS:
   --help, -h  show help

```

#### ctr containers create 创建容器

```
#ctr containers create 创建容器
#注意：ctr containers create只是创建了静态容器，此时容器还未启动运行
#使用ctr tasks start 启动已经创建的容器
USAGE:
   ctr containers create [command options] [flags] Image|RootFS CONTAINER [COMMAND] [ARG...]
OPTIONS:
   --snapshotter value               快照的名字。空值表示默认值。[$CONTAINERD_SNAPSHOTTER]
   --snapshotter-label value         为此容器的新快照添加的标签。
   --config value, -c value          指定runtime-specific的配置文件路径
   --cwd value                       指定进程的工作目录
   --env value                       指定环境变量 (e.g. FOO=bar)
   --env-file value                  指定环境环境变量文件，文件内容如：FOO=bar, 一行一个环境变量
   --label value                     指定格外的labels (e.g. foo=bar)
   --annotation value                指定额外的OCI annotations (e.g. foo=bar)
   --mount value                     指定额外的容器挂载(e.g. type=bind,src=/tmp,dst=/host,options=rbind:ro)
   --net-host                        容器启用主机网络
   --privileged                      运行特权容器
   --read-only                       将容器文件系统设置为只读
   --runtime value                   运行时名称 (default: "io.containerd.runc.v2")
   --runtime-config-path value       可选运行时配置路径
   --tty, -t                         为容器分配一个TTY
   --with-ns value                   指定要在容器运行时加入的现有Linux名称空间(format '<nstype>:<path>')
   --pid-file value                  task 的pid文件路径
   --gpus value                      向容器中添加gpu
   --allow-new-privs                 关闭OCI spec.NoNewPrivileges 特性
   --memory-limit value              容器的内存限制(以字节为单位) (default: 0)
   --device value                    要添加到容器的设备的文件路径;或者是要添加到容器的设备目录树的路径
   --cap-add value                   add Linux capabilities (Set capabilities with 'CAP_' prefix)
   --cap-drop value                  drop Linux capabilities (Set capabilities with 'CAP_' prefix)
   --seccomp                         启用默认的seccomp配置文件
   --seccomp-profile value           自定义seccomp配置文件的文件路径。在使用Seccomp -profile之前，Seccomp必须设置为true
   --apparmor-default-profile value  使用指定名称的默认配置文件启用AppArmor, e.g. "cri-containerd.apparmor.d"
   --apparmor-profile value          enable AppArmor with an existing custom profile
   --rdt-class value                 name of the RDT class to associate the container with. Specifies a Class of Service (CLOS) for cache and memory bandwidth management.
   --rootfs                          use custom rootfs that is not managed by containerd snapshotter
   --no-pivot                        disable use of pivot-root (linux only)
   --cpu-quota value                 限制CPU CFS配额 (default: -1)
   --cpu-period value                限制CPU CFS周期 (default: 0)
   --rootfs-propagation value        set the propagation of the container rootfs

示例：
#这就创建了一个名称叫做my-nginx的容器
#注意：ctr containers create只是创建了静态容器，此时容器还未启动运行
# 下面将讲解使用ctr tasks start 启动已经创建的容器
ctr containers create docker.io/library/nginx:1.18 my-nginx

```

#### ctr containers ls 列出容器

```
#ctr containers ls 用于查看/列出全部的容器，含启动和未启动
USAGE:
   ctr containers list [command options] [flags] [<filter>, ...]
[root@localhost ~]# ctr containers ls		#列出容器
CONTAINER    IMAGE                           RUNTIME                  
my-nginx     docker.io/library/nginx:1.18    io.containerd.runc.v2 

```

#### ctr containers info 获取容器信息

```
#查看容器的信息
[root@localhost ~]# ctr containers info my-nginx

```

#### ctr containers rm 删除容器

```
#删除一个或多个容器,delete、del、remove、rm都是容器
ctr containers delete
ctr containers del
ctr containers remove
ctr containers rm 
USAGE:
   ctr containers delete [command options] [flags] CONTAINER [CONTAINER, ...]
[root@localhost ~]# ctr containers rm my-nginx

```

### ctr tasks 管理任务

ctr tasks 命令用于管理任务。

```
#ctr tasks命令使用帮助，ctr tasks、ctr t、ctr task均等价。
[root@localhost ~]# ctr tasks --help
NAME:
   ctr tasks - manage tasks
USAGE:
   ctr tasks command [command options] [arguments...]
COMMANDS:
   attach                   attach to the IO of a running container
   checkpoint               checkpoint a container
   delete, del, remove, rm  删除任务
   exec                     进入容器（在容器里面执行进程）
   list, ls                 查看/列出任务
   kill                     发送信号给容器 (default: SIGTERM)
   pause                    暂停容器运行
   ps                       查看/列出容器进程
   resume                   恢复暂停的容器
   start                    启动已经创建的容器
   metrics, metric          使用内置的Linux运行时为任务获取metrics

OPTIONS:
   --help, -h  show help

```

#### ctr tasks start 启动已经创建的容器

```
[root@localhost ~]# ctr tasks start --help
NAME:
   ctr tasks start - start a container that has been created
USAGE:
   ctr tasks start [command options] CONTAINER
OPTIONS:
   --null-io         send all IO to /dev/null
   --log-uri value   log uri
   --fifo-dir value  directory used for storing IO FIFOs
   --pid-file value  file path to write the task's pid
   --detach, -d      后台运行，不用占用前台界面
[root@localhost ~]# 
#演示示例
# 启动刚才创建好的my-nginx
ctr tasks start -d my-nginx

```

#### ctr tasks list, ls 查看任务

```
[root@localhost ~]# ctr tasks ls --quite
Incorrect Usage: flag provided but not defined: -quite
NAME:
   ctr tasks list - list tasks
USAGE:
   ctr tasks list [command options] [flags]
OPTIONS:
   --quiet, -q  仅打印task id
 
 #演示示例
[root@localhost ~]# ctr tasks ls 		#查看任务
TASK        PID     STATUS    
my-nginx    9835    RUNNING
#示例
[root@localhost ~]# ctr tasks ls -q		#查看任务，仅打印任务ID
my-nginx
[root@localhost ~]# 

```

#### ctr tasks ps 查看 / 列出容器进程

```
USAGE:
   ctr tasks ps CONTAINER
示例：
[root@localhost ~]# ctr tasks ps my-nginx
PID     INFO
9835    -					#容器里面nginx的master进程ID
9872    -					#容器里面nginx的worker进程ID
[root@localhost ~]# 
#容器的进程其实就是宿主机的进程，因为容器对宿主机而言就是一个进程，如下查看宿主机的进程ID其实就是对应容器里面的进程ID：
[root@localhost ~]# ps -ef |grep nginx
root       9835   9814  0 01:21 ?        00:00:00 nginx: master process nginx -g daemon off;
101        9872   9835  0 01:21 ?        00:00:00 nginx: worker process
[root@localhost ~]# 

```

#### ctr tasks pause 暂停容器运行

```
[root@localhost ~]# ctr tasks pause --help
NAME:
   ctr tasks pause - pause an existing container
USAGE:
   ctr tasks pause CONTAINER
示例：
[root@localhost ~]#  ctr tasks pause my-nginx
[root@localhost ~]#  ctr tasks ls		#查看任务，已经是暂停状态
TASK        PID     STATUS    
my-nginx    9835    PAUSED
[root@localhost ~]# 

```

#### ctr tasks resume 恢复暂停的容器

```
[root@localhost ~]# ctr tasks resume  --help
NAME:
   ctr tasks resume - resume a paused container
USAGE:
   ctr tasks resume CONTAINER
[root@localhost ~]# 
示例：
[root@localhost ~]#  ctr tasks resume my-nginx
[root@localhost ~]#  ctr tasks ls		#查看任务，已经是运行状态
TASK        PID     STATUS    
my-nginx    9835    RUNNING
[root@localhost ~]# 

```

#### ctr tasks kill 发送信号给容器 (default: SIGTERM)

```
ctr tasks kill 命令其实就是停止容器，因为默认就是发送终端信号给容器。
[root@localhost ~]# ctr tasks kill --help
NAME:
   ctr tasks kill - signal a container (default: SIGTERM)
USAGE:
   ctr tasks kill [command options] [flags] CONTAINER
OPTIONS:
   --signal value, -s value  signal to send to the container
   --exec-id value           process ID to kill
   --all, -a                 send signal to all processes inside the container
[root@localhost ~]# 
[root@localhost ~]# ctr tasks kill my-nginx
[root@localhost ~]# ctr tasks ls	#容器已经处于停止状态了
TASK        PID     STATUS    
my-nginx    9835    STOPPED
[root@localhost ~]# 
#好像没有发现使用什么命令能使已经是停止状态的任务重启，使用ctr tasks start 报错任务已存在，也没有ctr tasks restart命令
#这个要先删除任务，ctr tasks del my-nginx,查看ctr containers ls 看到容器还在，直接ctr tasks start 启动任务接口

```

#### ctr tasks delete, del, remove, rm 删除任务

```
ctr tasks del默认删除的是处于停止状态的容器，可以使用-f参数强制删除运行状态的容器
[root@localhost ~]# ctr tasks delete --help
NAME:
   ctr tasks delete - delete one or more tasks
USAGE:
   ctr tasks delete [command options] CONTAINER [CONTAINER, ...]
OPTIONS:
   --force, -f      强制删除
   --exec-id value  process ID to kill
[root@localhost ~]# 
示例：
[root@localhost ~]# ctr tasks del my-nginx		#删除任务
[root@localhost ~]# ctr tasks ls				#没有任务
TASK    PID    STATUS    
[root@localhost ~]# ctr containers ls					#其实容器还在，只不过还没启动
CONTAINER    IMAGE                           RUNTIME                  
my-nginx     docker.io/library/nginx:1.18    io.containerd.runc.v2    
[root@localhost ~]# 

```

#### ctr tasks exec 进入容器

```
ctr tasks exec命令其实是在容器里面执行额外的进程，当前，我们习惯叫做进入容器
[root@localhost ~]# ctr tasks exec --help
NAME:
   ctr tasks exec - execute additional processes in an existing container
USAGE:
   ctr tasks exec [command options] [flags] CONTAINER CMD [ARG...]
OPTIONS:
   --cwd value       新进程的工作目录
   --tty, -t         在容器分配一个tty终端
   --detach, -d      detach from the task after it has started execution
   --exec-id value   执行进程的ID,必须指定，可以随意指定，也可以使用随机变量$RANDOM自动生成
   --fifo-dir value  用于存储IO fifo的目录
   --log-uri value   用于自定义shim日志的日志uri
   --user value      用户id或名称
[root@localhost ~]# 
示例：
[root@localhost ~]# ctr tasks exec -t --exec-id $RANDOM my-nginx sh
[root@localhost ~]# ctr tasks exec --exec-id $RANDOM my-nginx ls -l

```

### ctr run 直接运行容器

上面使用的 ctr containers create 创建的是一个静态容器，还要使用 ctrtasks start 命令启动静态容器，这里我们可以直接使用 ctr run 命令直接运行容器。

```
[root@localhost ~]# ctr run --help
NAME:
   ctr run - run a container
USAGE:
   ctr run [command options] [flags] Image|RootFS ID [COMMAND] [ARG...]
OPTIONS:
   --rm                                    容器退出后即可删除容器, 不能与--detach参数一起使用
   --null-io                               将全部的输入输出发送到/dev/null
   --log-uri value                         日志的uri
   --detach, -d                            后台运行，不能与--rm参数一起使用
   --fifo-dir value                        directory used for storing IO FIFOs
   --cgroup value                          cgroup path (To disable use of cgroup, set to "" explicitly)
   --platform value                        run image for specific platform
   --cni                                   enable cni networking for the container
   --runc-binary value                     specify runc-compatible binary
   --runc-root value                       specify runc-compatible root
   --runc-systemd-cgroup                   start runc with systemd cgroup manager
   --uidmap container-uid:host-uid:length  run inside a user namespace with the specified UID mapping range; specified with the format container-uid:host-uid:length
   --gidmap container-gid:host-gid:length  run inside a user namespace with the specified GID mapping range; specified with the format container-gid:host-gid:length
   --remap-labels                          provide the user namespace ID remapping to the snapshotter via label options; requires snapshotter support
   --cpus value                            set the CFS cpu quota (default: 0)
   --cpu-shares value                      set the cpu shares (default: 1024)
   --snapshotter value                     snapshotter name. Empty value stands for the default value. [$CONTAINERD_SNAPSHOTTER]
   --snapshotter-label value               labels added to the new snapshot for this container.
   --config value, -c value                path to the runtime-specific spec config file
   --cwd value                             specify the working directory of the process
   --env value                             specify additional container environment variables (e.g. FOO=bar)
   --env-file value                        specify additional container environment variables in a file(e.g. FOO=bar, one per line)
   --label value                           specify additional labels (e.g. foo=bar)
   --annotation value                      specify additional OCI annotations (e.g. foo=bar)
   --mount value                           specify additional container mount (e.g. type=bind,src=/tmp,dst=/host,options=rbind:ro)
   --net-host                              容器启用宿主机网络
   --privileged                            运行特权容器
   --read-only                             设置容器的文件系统为只读
   --runtime value                         运行时名称 (default: "io.containerd.runc.v2")
   --runtime-config-path value             optional runtime config path
   --tty, -t                               分配一个TTY 终端
   --with-ns value                         specify existing Linux namespaces to join at container runtime (format '<nstype>:<path>')
   --pid-file value                        file path to write the task's pid
   --gpus value                            add gpus to the container
   --allow-new-privs                       turn off OCI spec's NoNewPrivileges feature flag
   --memory-limit value                    memory limit (in bytes) for the container (default: 0)
   --device value                          file path to a device to add to the container; or a path to a directory tree of devices to add to the container
   --cap-add value                         add Linux capabilities (Set capabilities with 'CAP_' prefix)
   --cap-drop value                        drop Linux capabilities (Set capabilities with 'CAP_' prefix)
   --seccomp                               enable the default seccomp profile
   --seccomp-profile value                 file path to custom seccomp profile. seccomp must be set to true, before using seccomp-profile
   --apparmor-default-profile value        enable AppArmor with the default profile with the specified name, e.g. "cri-containerd.apparmor.d"
   --apparmor-profile value                enable AppArmor with an existing custom profile
   --rdt-class value                       name of the RDT class to associate the container with. Specifies a Class of Service (CLOS) for cache and memory bandwidth management.
   --rootfs                                use custom rootfs that is not managed by containerd snapshotter
   --no-pivot                              disable use of pivot-root (linux only)
   --cpu-quota value                       Limit CPU CFS quota (default: -1)
   --cpu-period value                      Limit CPU CFS period (default: 0)
   --rootfs-propagation value              set the propagation of the container rootfs  
[root@localhost ~]# 
示例：
[root@localhost ~]# ctr run -d docker.io/library/nginx:1.18 my-nginx2
[root@localhost ~]# ctr tasks ls
TASK         PID      STATUS    
my-nginx2    10896    RUNNING
[root@localhost ~]# 

#哎，没有端口映射的吗，没有port参数呀，ctr containers create也是没有port相关的参数

```