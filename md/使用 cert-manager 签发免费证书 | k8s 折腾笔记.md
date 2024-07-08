> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [todoit.tech](https://todoit.tech/k8s/cert/)

> k8s 折腾笔记

**前置条件：**

*   [在 ubuntu 虚拟机安装 k8s 集群](https://todoit.tech/k8s/install/k8s.html)
*   [本地 k8s 集群也可以有 LoadBalancer](https://todoit.tech/k8s/mentallb/)
*   [从 Ingress 开始有了域名](https://todoit.tech/k8s/ingress/nginx/)

使用 HTTPS 需要向权威机构申请证书，并且需要付出一定的成本，如果需求数量多，则开支也相对增加。[cert-manageropen in new window](https://cert-manager.io/docs/) 是 Kubernetes 上的全能证书管理工具，支持利用 cert-manager 基于 [ACMEopen in new window](https://tools.ietf.org/html/rfc8555) 协议与 [Let's Encryptopen in new window](https://letsencrypt.org/) 签发免费证书并为证书自动续期，实现永久免费使用证书。

本地虚拟机 k8s 集群，也是可以使用 cert-manager 和 Let's Encrypt 组合的。

前提是，你有一个属于你自己的域名。比如 [todoit.techopen in new window](https://todoit.tech/) 就是作者本人的域名。

[#](#安装-cert-manager) 安装 cert-manager
-------------------------------------

cert-manager 将证书和证书颁发者作为资源类型添加到 Kubernetes 集群中，并简化了获取、更新和使用这些证书的过程。

它可以从各种受支持的来源颁发证书，包括 Let’s Encrypt、HashiCorp Vault 和 Venafi 以及私有 PKI。

它将确保证书有效且是最新的，并在到期前的配置时间尝试更新证书。

![](https://todoit.oss-cn-shanghai.aliyuncs.com/todoit/README-2021-10-14-20-29-31.svg)

上图显示，cert-manager 拥有 Issuer 和 Certificate 等自定义资源，我们稍后会创建这些资源。

添加 cert-manager 仓库

```
helm repo add jetstack https://charts.jetstack.io
helm repo update


```

生成 values.yaml

```
helm show values jetstack/cert-manager > values.yaml


```

修改 values.yaml

```
installCRDs: true

prometheus:
  enabled: false

webhook:
  timeoutSeconds: 10


```

如果想查看生成的清单，可以使用

```
helm template cert-manager jetstack/cert-manager -n cert-manager -f values.yaml > cert-manager.yaml


```

安装 cert-manager

```
helm install cert-manager jetstack/cert-manager -n cert-manager --create-namespace -f values.yaml


```

等待

```
kubectl wait --for=condition=Ready pods --all -n cert-manager






```

[#](#创建-issuer) 创建 Issuer
-------------------------

Let’s Encrypt 利用 ACME (Automated Certificate Management Environment) 协议校验域名的归属，校验成功后可以自动颁发免费证书。免费证书有效期只有 90 天，需在到期前再校验一次实现续期。使用 cert-manager 可以自动续期，即实现永久使用免费证书。校验域名归属的两种方式分别是 HTTP-01 和 DNS-01，校验原理详情可参见 [Let's Encrypt 的运作方式 open in new window](https://letsencrypt.org/zh-cn/how-it-works/)。

DNS-01 校验支持泛域名， 但是是不同 DNS 提供商的配置方式不同，DNS 提供商过多而 cert-manager 的 Issuer 不能全部支持。部分可以通过部署实现 cert-manager 的 Webhook 服务来扩展 Issuer 进行支持。例如阿里 DNS 就是通过 Webhook 的方式进行支持。

由于作者在编写本文时，未能找到能在本地虚拟机 k8s 集群正常工作的阿里 DNS Webhook，所以将域名迁移至 [cloudflareopen in new window](https://www.cloudflare.com/zh-cn/)。或许待作者日后有能力自己写一个。

> 关于如何将域名迁移到 cloudflare，请参考其它资料。

添加 DNS 记录，服务器可以用阿里云的

![](https://todoit.oss-cn-shanghai.aliyuncs.com/todoit/README-2021-10-15-20-19-26.jpg)

前往个人资料 -> API 令牌， 使用编辑区域 DNS 模版， 创建一个 token。

![](https://todoit.oss-cn-shanghai.aliyuncs.com/todoit/README-2021-10-15-20-28-14.jpg)

测试令牌是否能正常工作：

```
curl -X GET "https://api.cloudflare.com/client/v4/user/tokens/verify" \
     -H "Authorization: Bearer <yout api token>" \
     -H "Content-Type:application/json"


```

作者将得到的 token，保存到 .env 文件中，并将 .env 添加到 .gitignore 中，毕竟这是敏感信息呢。

```
api-token=Anbv2XXQ4pOqKYHGLtx5uSnnAqzTkvCPzxogjLKP


```

我们使用 [Kustomizeopen in new window](https://kubernetes.io/zh/docs/tasks/manage-kubernetes-objects/kustomization/) 来管理该 token。

编写 kustomization.yaml

```
resources:
  - letsencrypt-issuer.yaml
namespace: cert-manager
secretGenerator:
  - name: cloudflare-api-token-secret
    envs:
      - .env 
generatorOptions:
  disableNameSuffixHash: true


```

我们使用 Kustomize 来生成一个名为 cloudflare-api-token-secret 的 Secret, 该 Secret 被下面的清单使用

```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-dns01
spec:
  acme:
    privateKeySecretRef:
      name: letsencrypt-dns01
    server: https://acme-v02.api.letsencrypt.org/directory
    solvers:
      - dns01:
          cloudflare:
            email: listenzz@163.com 
            apiTokenSecretRef:
              key: api-token
              name: cloudflare-api-token-secret 


```

所生成的 Secret 可以使用下面的命令来检查

如无问题，运行如下命令，创建 issuer

查看 Let't Encrypt 注册状态

```
kubectl describe clusterissuer letsencrypt-dns01


```

如下表示注册成功

![](https://todoit.oss-cn-shanghai.aliyuncs.com/todoit/README-2021-10-15-18-57-22.png)

[#](#创建-certificate) 创建 Certificate
-----------------------------------

```
kubectl apply -f certificate.yaml


```

```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: wildcard-todoit-tech
  namespace: default
spec:
  dnsNames:
    - "*.todoit.tech" 
  issuerRef:
    kind: ClusterIssuer
    name: letsencrypt-dns01 
  secretName: wildcard-letsencrypt-tls 


```

查看证书是否签发成功

```
kubectl get certificate -w








```

注意事项：Let's Encrypt 一个星期内只为同一个域名颁发 5 次证书，todoit.tech 和 whoami.todoit.tech 被视为不同的域名。

可以通过以下方式来了解为何未能颁发成功的原因：

![](https://todoit.oss-cn-shanghai.aliyuncs.com/todoit/README-2021-10-15-19-01-57.png)

[#](#测试) 测试
-----------

```
kubectl apply -f whoami.yaml


```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami
  labels:
    app: containous
    name: whoami
spec:
  replicas: 2
  selector:
    matchLabels:
      app: containous
      task: whoami
  template:
    metadata:
      labels:
        app: containous
        task: whoami
    spec:
      containers:
        - name: containouswhoami
          image: containous/whoami
          resources:
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: whoami
spec:
  ports:
    - name: http
      port: 80
  selector:
    app: containous
    task: whoami
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: whoami-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
    - hosts:
        - "whoami.todoit.tech"
      secretName: wildcard-letsencrypt-tls
  rules:
    - host: whoami.todoit.tech
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: whoami
                port:
                  number: 80


```

![](https://todoit.oss-cn-shanghai.aliyuncs.com/todoit/README-2021-10-15-19-11-36.png)

在浏览器中输入 whoami.todoit.tech，可以看到由 R3 给我们颁发了一张合法的证书

![](https://todoit.oss-cn-shanghai.aliyuncs.com/todoit/README-2021-10-15-19-25-48.jpg)

* * *

**参考**

*   [使用 Helm 安装 cert-manageropen in new window](https://cert-manager.io/docs/installation/helm/)
*   [配置使用 Cloudflareopen in new window](https://cert-manager.io/docs/configuration/acme/dns01/cloudflare/)