> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/GB0Iwdh2YUnJiXA1gLp9TA?poc_token=HKqAIGajiwbEcZ-6-HAq_sF4tO2r9HP1JQ-ZLkxd)

背景
--

**配置中心问题：**

对于在云原生中配置中心，例如 configmap 和 secret 对象，虽然可以进行直接更新资源对象

*   对于引用这些有些不变的配置是可以打包到镜像中的，那可变的配置呢？
    
*   信息泄漏，很容易引发安全风险，尤其是一些敏感信息，比如密码、密钥等。
    
*   每次配置更新后，都要重新打包一次，升级应用。镜像版本过多，也给镜像管理和镜像中心存储带来很大的负担。
    
*   定制化太严重，可扩展能力差，且不容易复用。
    

**使用方式：**

Configmap 或 Secret 使用有两种方式，一种是 env 系统变量赋值，一种是 volume 挂载赋值，env 写入系统的 configmap 是不会热更新的，而 volume 写入的方式支持热更新！

*   对于 env 环境的，必须要滚动更新 pod 才能生效，也就是删除老的 pod，重新使用镜像拉起新 pod 加载环境变量才能生效。
    
*   对于 volume 的方式，虽然内容变了，但是需要我们的应用直接监控 configmap 的变动，或者一直去更新环境变量才能在这种情况下达到热更新的目的。
    
*   应用不支持热更新，可以在业务容器中启动一个 sidercar 容器，监控 configmap 的变动，更新配置文件，或者也滚动更新 pod 达到更新配置的效果。
    

解决方案
----

ConfigMap 和 Secret 是 Kubernetes 常用的保存配置数据的对象，你可以根据需要选择合适的对象存储数据。通过 Volume 方式挂载到 Pod 内的，kubelet 都会定期进行更新。但是通过环境变量注入到容器中，这样无法感知到 ConfigMap 或 Secret 的内容更新。

目前如何让 Pod 内的业务感知到 ConfigMap 或 Secret 的变化，还是一个待解决的问题。但是我们还是有一些 Workaround 的。

如果业务自身支持 reload 配置的话，比如 nginx -s reload，可以通过 inotify 感知到文件更新，或者直接定期进行 reload（这里可以配合我们的 readinessProbe 一起使用）。

如果我们的业务没有这个能力，考虑到不可变基础设施的思想，我们是不是可以采用滚动升级的方式进行？没错，这是一个非常好的方法。目前有个开源工具 Reloader，它就是采用这种方式，通过 watch ConfigMap 和 Secret，一旦发现对象更新，就自动触发对 Deployment 或 StatefulSet 等工作负载对象进行滚动升级。

reloader 简介
-----------

**reloader 简介：**

Reloader 可以观察 ConfigMap 和 Secret 中的变化，并通过相关的 deploymentconfiggs、 deploymentconfiggs、 deploymonset 和 statefulset 对 Pods 进行滚动升级。

**reloader 安装：**

helm 安装：

```
helm repo add stakater https://stakater.github.io/stakater-charts

helm repo update

helm install stakater/reloader


```

Kustomize：

```
kubectl apply -k https://github.com/stakater/Reloader/deployments/kubernetes


```

资源清单安装：

```
kubectl apply -f https://raw.githubusercontent.com/stakater/Reloader/master/deployments/kubernetes/reloader.yaml
# 在此安装在common-service 名称空间下，
[root@master reloader]# kubectl apply -f reloader.yaml 
clusterrole.rbac.authorization.k8s.io/reloader-reloader-role created
clusterrolebinding.rbac.authorization.k8s.io/reloader-reloader-role-binding created
deployment.apps/reloader-reloader created
serviceaccount/reloader-reloader created
[root@master reloader]# kubectl get all -n common-service 
NAME                                     READY   STATUS    RESTARTS   AGE
pod/reloader-reloader-66d46d5885-nx64t   1/1     Running   0          15s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/reloader-reloader   1/1     1            1           16s

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/reloader-reloader-66d46d5885   1         1         1       16s


```

配置忽略:

reloader 能够配置忽略 cm 或者 secrets 资源，可以通过配置在 reader deploy 中的 spec.template.spec.containers.args，如果两个都忽略，那就缩小 deploy 为 0，或者不部署 reoader。

<table><thead><tr><th>Args</th><th>Description</th></tr></thead><tbody><tr><td>–resources-to-ignore=configMaps</td><td>To ignore configMaps</td></tr><tr><td><br></td><td><br></td></tr><tr><td>–resources-to-ignore=secrets</td><td>To ignore secrets</td></tr><tr><td><br></td><td><br></td></tr></tbody></table>

**配置：**

自动更新：

reloader.stakater.com/search 和 reloader.stakater.com/auto 并不在一起工作。如果你在你的部署上有一个 reloader.stakater.com/auto : “true” 的注释，该资源对象引用的所有 configmap 或这 secret 的改变都会重启该资源，不管他们是否有 reloader.stakater.com/match : “ true” 的注释。

```
kind: Deployment
metadata:
  annotations:
    reloader.stakater.com/auto: "true"
spec:
  template: metadata:


```

制定更新：

指定一个特定的 configmap 或者 secret，只有在我们指定的配置图或秘密被改变时才会触发滚动升级，这样，它不会触发滚动升级所有配置图或秘密在部署，后台登录或状态设置中使用。

一个制定 deployment 资源对象，在引用的 configmap 或者 secret 种，只有`reloader.stakater.com/match: "true"`为 true 才会出发更新，为 false 或者不进行标记，该资源对象都不会监视配置的变化而重启。

```
kind: Deployment
metadata:
  annotations:
    reloader.stakater.com/search: "true"
spec:
  template:


```

cm 配置：

```
kind: ConfigMap
metadata:
  annotations:
    reloader.stakater.com/match: "true"
data:
  key: value


```

指定 cm：

如果一个 deployment 挂载有多个 cm 或者的场景下，我们只希望更新特定一个 cm 后，deploy 发生滚动更新，更新其他的 cm，deploy 不更新，这种场景可以将 cm 在 deploy 中指定为单个或着列表实现。

例如：一个 deploy 有挂载 nginx-cm1 和 nginx-cm2 两个 configmap，只想 nginx-cm1 更新的时候 deploy 才发生滚动更新，此时无需在两个 cm 中配置注解，只需要在 deploy 中写入 configmap.reloader.stakater.com/reload:nginx-cm1，其中 nginx-cm1 如果发生更新，deploy 就会触发滚动更新。

如果多个 cm 直接用逗号隔开

```
# configmap对象
kind: Deployment
metadata:
  annotations:
    configmap.reloader.stakater.com/reload: "nginx-cm1"
spec:
  template: metadata:


```

```
# secret对象
kind: Deployment
metadata:
  annotations:
    secret.reloader.stakater.com/reload: "foo-secret"
spec:
  template: metadata:


```

无需在 cm 或 secret 中添加注解，只需要在引用资源对象中添加注解即可。

测试验证
----

**deploy：**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
   # reloader.stakater.com/auto: "true"
    reloader.stakater.com/search: "true"
  labels:
    run: nginx
  name: nginx
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        # 必须匹配volumes的名称,定义configmap
        - name: nginx-cm
          mountPath: /data/cfg
          readOnly: true
      volumes:
      # 定义逻辑卷的名称
      - name: nginx-cm
        configMap:
          # 使用configmap资源的名称
          name: nginx-cm
          items:
          # 使用configmap中到那个key
          - key: config.yaml
            # 使用configmap中到key映射到容器中到文件名称
            path: config.yaml
            mode: 0644 


```

**configmap：**

```
apiVersion: v1
data:
  config.yaml: |
    # project settings

    # go2cloud_api service config
    DEFAULT_CONF:
      port: 8888
    # data disk api
    UNITTEST_TENCENT_ZONE: ap-chongqing-1
kind: ConfigMap
metadata:
  name: nginx-cm
  annotations:
    reloader.stakater.com/match: "true"


```

**测试：**

```
[root@master ns-default]# kubectl  get po
NAME                     READY   STATUS    RESTARTS   AGE
nginx-68c9bf4ff7-9gmg6   1/1     Running   0          10m
[root@master ns-default]# kubectl  get cm
NAME       DATA   AGE
nginx-cm   1      28m
# 更新cm内容
[root@master ns-default]# kubectl edit cm nginx-cm 
configmap/nginx-cm edited
# 查看po发生了滚动更新，重新加载配置文件
[root@master ns-default]# kubectl get po
NAME                     READY   STATUS              RESTARTS   AGE
nginx-66c758b548-9dllm   0/1     ContainerCreating   0          4s
nginx-68c9bf4ff7-9gmg6   1/1     Running             0          10m


```

注意事项
----

*   reloader 为全局资源对象，建议部署在一个公共服务的 ns 下，然后其他 ns 也可以正常使用 reloader 特性。
    
*   Reloader.stakater.com/auto : 如果配置 configmap 或者 secret 在 deploymentconfigmap/deployment/daemonsets/Statefulsets
    
*   secret.reloader.stakater.com/reload 或者 configmap.reloader.stakater.com/reload 注释中被使用，那么 true 只会重新加载 pod，不管使用的是 configmap 还是 secret。
    
*   reloader.stakater.com/search 和 reloader.stakater.com/auto 不能同时使用。如果你在你的部署上有一个 reloader.stakater.com/auto : “true” 的注释，那么它总是会在你修改了 configmaps 或者使用了机密之后重新启动，不管他们是否有 reloader.stakater.com/match : “ true” 的注释。
    

反思
--

Reloader 通过 watch ConfigMap 和 Secret，一旦发现对象更新，就自动触发对 Deployment 或 StatefulSet 等工作负载对象进行滚动升级。

如果我们的应用内部没有去实时监控配置文件，利用该方式可以非常方便的实现配置的热更新。

参考链接
----

*   https://github.com/stakater/Reloader
    

来源 (版权归原作者所有)：https://bbs.huaweicloud.com/blogs/312634

**添加👇下面微信，拉你进群与大佬一起探讨云原生！**

![](https://mmbiz.qpic.cn/mmbiz_jpg/RxCylSQYVE9psKrdVlptu7HIHGIVrsEgNEIGlZiaKhTScVZMkQMCfcgalfSpOmtuf32uCpKTmczSBSaV066nyFw/640?wx_fmt=jpeg&from=appmsg)