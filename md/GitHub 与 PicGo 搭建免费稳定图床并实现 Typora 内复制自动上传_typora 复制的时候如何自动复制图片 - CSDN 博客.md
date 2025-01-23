> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/zhebushibiaoshifu/article/details/129455228)

  本文介绍基于 **Github** 平台与 **PicGo** 工具，构建免费、稳定的**图床**，并实现在 **Typora** 内撰写 **Markdown** 文档时，粘贴图片就可以将这一图片**自动上传**到搭建好的[图床](https://so.csdn.net/so/search?q=%E5%9B%BE%E5%BA%8A&spm=1001.2101.3001.7020)中的方法。

1 配置 GitHub
-----------

  首先，我们需要配置 **Github**，创建一个仓库从而保存我们的图片。

  进入 **Github** 平台的[官方网站](https://github.com/)。注册或登录账号后，点击屏幕左侧的 “**New**” 按钮，从而新建一个仓库（**Repository**），如下图所示。这个 **Repository** 就是我们后期图片的保存位置。

![](https://i-blog.csdnimg.cn/blog_migrate/4ea937edfde9d77dbdb43603a22a6bb0.png)

  随后，在如下图所示的 **Repository** 配置界面中，配置 **Repository** 的信息。其中，下图两个紫色框内的内容是大家 **Repository** 的**名称**与简介，因为我们是配置图床，所以就可以写一些和图片有关的名称与简介即可（但要注意**名称**中不要含有空格或特殊字符）。随后，要确保在下图所示的红色框内选择”**Public**“，否则之后我们在外部访问我们图床中的图片，就会由于没有权限导致失败。此外，其他的信息大家就随意选择即可，建议保持默认。

![](https://i-blog.csdnimg.cn/blog_migrate/9d38ac6553d23ff12f8a1ad8b86704cd.png)

  随后，选择”**Creat repository**“即可。接下来，在页面右上角，点开我们的头像，并选择”**Settings**“，如下图所示。

![](https://i-blog.csdnimg.cn/blog_migrate/2aa5d632b816d172ff3d41ae0b27cc75.png)

  随后，选择其中左下角的”**Developer settings**“选项，如下图所示。

![](https://i-blog.csdnimg.cn/blog_migrate/71abcedb58ebc63c6e5969232f2313fc.png)

  随后，选择”**Personal access tokens**“，并选择其中下方的”**Tokens (classic)**”；随后，选择右上角”**Generate new token**“，并再选择”**Generate new token (classic)**“。如下图所示。

![](https://i-blog.csdnimg.cn/blog_migrate/3632e3fe005d8e8d02a2da7b1bcfb435.png)

  随后，在弹出的配置界面中，首先填写”**Note**“，这个是当前 **token** 的**注释**，也用一个和图片有关的名称来填写即可；随后，配置 **token** 的有效期。其实我们可以将有效期选择为**永久**，但是 **GitHub** 官方强烈不推荐这种**永久**期限的 **token**，因此可以选择`90`天，之后过期了我们继续来设置新的有效期就好。随后，配置勾选项，我这里是将全部的勾选项都选中了，但是其实只要保证`repo`开头的勾选项选中即可。

![](https://i-blog.csdnimg.cn/blog_migrate/0343bb5545ec22b694c27ea8451cf756.png)

  接下来，即可看到此时 **token** 的**序号**已经获取了，如下图所示。这里大家一定需要保存一下当前的**序号**，之后就看不到这个**序号**了。

![](https://i-blog.csdnimg.cn/blog_migrate/0603166fa675dbdb87f93088e4636781.png)

  至此，我们就完成了 **GitHub** 上的配置操作。

2 配置 PicGo
----------

  接下来，我们需要配置 **PicGo**。**PicGo** 是一个工具，从而将我们的图片上传到 **GitHub** 中。

  同样的，我们还是直接进入 **PicGo** 的[官方网站](https://picgo.github.io/PicGo-Doc/zh/guide/#picgo-is-here)，如下图所示。

![](https://i-blog.csdnimg.cn/blog_migrate/1edf3f64f2576de751223e7dddafde56.png)

  随后，下拉找到其下载地址，并选择一个进行下载。

![](https://i-blog.csdnimg.cn/blog_migrate/329f7a308d44d8862ed868f5489cfacd.png)

  例如，我这里就在 **GitHub** 进行下载。由于我是 **Windows** 操作系统的电脑，因此选择下图紫色框内所示的安装包即可。

![](https://i-blog.csdnimg.cn/blog_migrate/13f58b23c69f5d414dd1b30b74f41ca9.png)  
  随后，安装 **PicGo** 并打开，如下图所示。其中，我们需要在” **图床设置** “中找到”**GitHub**“，并配置各项信息。其中，仓库名就是我们前面创建的 **Repository** 的名称，分支名很多博主是用的`master`，如下图所示，但是我这里这么设置有问题（下文会提到）；随后的 **Token** 就是前面我们获取的 **token** 序号，存储路径这里我们可以空着，如果大家需要指定将图片存储到仓库中的某个路径下，就在这里设置即可。随后的自定义域名，大家可以填写`https://cdn.jsdelivr.net/gh/Chutj/Pictures@master`，这样可以在使用图床时获取一定加速，但需要注意将其中的**仓库名**部分修改为大家自己的仓库名称。

![](https://i-blog.csdnimg.cn/blog_migrate/8ecb9c26991835636856a149ef6398f7.png)

  前面提到我们分支名的填写，这里应该是由于 **GitHub** 网站的调整，仓库的默认分支名称修改为了`main`，如下图所示。

![](https://i-blog.csdnimg.cn/blog_migrate/231efc8b6fd56b53fc21a6b62e020daf.png)

  因此，我这里也需要在 **PicGo** 中调整分支名为`main`，如下图所示。大家在设置时，可以到 **GitHub** 中确认一下再填写。

![](https://i-blog.csdnimg.cn/blog_migrate/cc4170db62da587a5e6b541e7f0c3428.png)

  随后，在”**PicGo 设置** “中，可以对快速上传图片的快捷键加以编辑。例如，我将第一个默认的快捷键加以调整，这一快捷键可以使得我们将剪切板中第一张图片自动上传到图床中。

![](https://i-blog.csdnimg.cn/blog_migrate/e8bc873298e336a5c220030108336eef.png)

  接下来，我们可以上传图片来试一下我们配置。注意，如果出现如下图所示的” **上传失败** “报错，证明我们的配置等可能有问题。

![](https://i-blog.csdnimg.cn/blog_migrate/d92b6c4b83ab3838c907884adfa1fe14.png)

  针对这一情况，我们可以打开” **设置日志文件** “，并在此打开日志。

![](https://i-blog.csdnimg.cn/blog_migrate/13e247003f557dec7ec92f4c12817967.png)

  随后，找到刚刚失败的记录，查看其中具体报错的内容。例如，我这里是因为一开始没有意识到 **GitHub** 的分支名称有所修改，所以导致的上传失败。

![](https://i-blog.csdnimg.cn/blog_migrate/59c1451be87c465b54fadc58ca313d86.png)

  上传成功图片后，我们就可以在 **GitHub** 指定的仓库中找到我们上传的图片。

  至此，我们就搭建好了自己免费、稳定的图床。如果大家只是需要构建图床，那么看到这里就完成全部的操作了。

3 配置 Typora
-----------

  但是，我还希望在 **Typora** 软件中，复制剪切板的图片，或者上传本地的图片后，自动将图片上传至前面配置好的图床中。因此，还需要配置一下 **Typora** 软件。

  首先，如果是第一次使用，我们需要下载一个 **Typora** 软件。可以选择下载正版软件，也可以用网上一些可以直接使用的版本的安装包。下载软件后安装并打开，在” **文件** “中选择” **偏好设置…**“，随后选择” **图像** “，并按照如下图所示的配置方式来加以设置。

![](https://i-blog.csdnimg.cn/blog_migrate/b28559f49e3fbec5742c0c582b4f1495.png)

  随后，选择上图中的” **验证图片上传选项** “，如果出现如下图所示的窗口，则表明我们已经成功配置完毕。

![](https://i-blog.csdnimg.cn/blog_migrate/95cfe122236e779d3e531fd5aae2fc0e.png)

  此时，如果我们在 **Typora** 内添加了图片，那么这张图片将自动通过 **PicGo**，上传到我们前面建立好的图床中。

  至此，大功告成。

欢迎关注：疯狂学习 GIS