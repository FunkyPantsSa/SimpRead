> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.51cto.com](https://blog.51cto.com/u_12227/10326827)

> git 查看主要的 realese 版本 git 查看当前的版本命令，1. 查看当前 git 版本（判断是否安装过 git）git--version2.git 下载代码 gitclonehttp://gitlab.te......

### git 查看主要的 realese 版本 git 查看当前的版本命令

1. 查看当前 git 版本（判断是否安装过 git）

2.git 下载代码

```
git clone http://gitlab.tech.xxx.com/xxx/backend-view.git

```

 3. 修改编辑项目常用

```
// 查看当前仓库文件状态（常在提交文件之前查看，会显示新增文件删除文件，已修改文件等状态）
git status

// 添加文件
git add .    // 添加所有已修改文件
git add fileName   // 添加指定文件名的文件（可在git status返回中复制）

// 提交修改说明
git commit -m "修改的内容"    // 记录当前提交的主题 以便区分每次提交的内容

// 拉取代码
git pull    // 拉取代码  push之前pull一次代码  （尤其多人开发一定注意push之前先pull）
git pull origin <远程分支名>     // 将远程指定分支 拉取到 本地当前分支上
git pull origin <远程分支名>:<本地分支名>     // 将远程指定分支 拉取到 本地指定分支上
git pull origin    // 将与本地当前分支同名的远程分支 拉取到 本地当前分支上(需先关联远程分支)

// 推送代码
git push    // 推送代码到远程仓库
git push origin <本地分支名>    // 将本地当前分支 推送到 与本地当前分支同名的远程分支上（注意：pull是远程在前本地在后，push相反）
git push origin <本地分支名>:<远程分支名>     // 将本地当前分支 推送到 远程指定分支上（注意：pull是远程在前本地在后，push相反）
git push origin     // 将本地当前分支 推送到 与本地当前分支同名的远程分支上(需先关联远程分支)

git push --set-upstream origin      // <本地分支名>将本地分支与远程同名分支相关联

```

 4. 分支相关

```
git branch      // 查看本地分支（名称前面加* 号的是当前的分支）
git branch -a      // 查看分支，远程分支会用红色表示出来（如果你开了颜色支持的话）
git branch -vv      // 查看本地分支和远程分支对应关系
git remote      // 列出所有远程主机
git remote update origin --prune      // 更新远程主机origin（gitlab有新分支，本地查看分支无法查看到的时候使用）
git branch -r       // 列出远程分支
git branch -vv       // 查看本地分支和远程分支对应关系
git checkout -b dev origin/dev      // 新建本地分支dev与远程dev分支相关联

```

         1 新建分支相关

```
git checkout -b newBranch
git push origin newBranch:newBranch    // 把新建的本地分支push到远程服务器，远程分支与本地分支同名（当然可以随意起名）：

```

        2 删除分支相关  
                1 删除本地分支 

```
git branch -D newBranch    // 删除本地 newBranch 分支
git checkout newBranch    // 如果需要重新拉取远程的newBranch分支 执行

```

                2 删除远程分支

```
git push origin :delBranchName
git push origin --delete delBranchName

```

```
// 切换分支,分支跟踪， 本地分支和远程分支的关系
git branch branchName     // 创建分支
git checkout branchName     // 切换分支
git branch -d branchName     // 删除本地分支

git branch -r -d origin/branch-name  
git push origin :branch-name     // 删除远程分支

// 如果远程新建了一个分支，本地没有该分支,git checkout --track origin/ branchName ，这时本地会新建一个分支名叫  branchName，会自动跟踪远程的同名分支 branchName。
git checkout --track origin/branchName

// 如果本地新建了一个分支 branchName，但是在远程没有。这时候 push 和 pull 指令就无法确定该跟踪谁，一般来说我们都会使其跟踪远程同名分支，所以可以利用 git push --set-upstream origin branchName ，这样就可以自动在远程创建一个 branchName 分支，然后本地分支会 track 该分支。后面再对该分支使用 push 和 pull 就自动同步。
git push --set-upstream origin branchName

// 合并分支（多人开发中，经常一人一个分支，各自在自己分支开发，开发完成以后合并到某一个指定分支，没有问题后最后合并到master主分支，我们的流程是各自在自己的develop开发，开发完成以后合并到lastest分支，没有问题后提交合并申请到master分支，由leader审批是否统一合并到master，因为很多新人不太清楚代码的具体用途，所以讲的稍微详细点，明白命令的实现目的能更好的掌握使用，后面会有具体的操作流程）
1.本地代码依次
git status
git add
git commit -m ""
git pull
git push （develop-author分支，即自己的开发分支）
以后（把本地代码推送到远程对应分支）
2.git checkout lastest （切换到lastest分支）
3.git pull origin lastest  （先把远程lastest分支修改内容拉取，多人开发，需要把远程lastest上的代码pull下来）
4.git  merge develop-author   （合并自己的分支到lastest）

```

本文章为转载内容，我们尊重原作者对文章享有的著作权。如有内容错误或侵权问题，欢迎原作者联系我们进行内容更正或删除文章。

**相关文章**