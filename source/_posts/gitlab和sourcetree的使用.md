title: gitlab和sourcetree的使用
date: 2016-01-19 00:34:21
tags: git
---

## 概述

网上关于git资料一堆一堆的，我们就不做git相关介绍，建议初学git的同学阅读下面两个文档
* git初学者教程[博客](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)  
* git官方文档[git](https://git-scm.com/)  
* 最广泛使用的分支管理模型[git-flow介绍](http://fann.im/blog/2012/03/12/git-flow-notes/)



在学习git之前，请全部忘掉你的svn，建立完整的分支思维。

本文只做安装和部分操作介绍，不做具体内容展开，学习最高效的手段永远是自己`练习，练习，练习`。


---

## 开始

常用的代码仓库托管网站有`github`和`gitlab`,下面首先从gitlab开始


## 安装

1. mac下的安装，打开终端，输入下面
	```shell
	brew install git
	brew install git-flow
	```
	如果没有brew，先安装
	```
		/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
	```

	mac下需安装基本开发环境
	```
	xcode-select --install
	```

2. windows下的安装

	点击下面地址下载安装。
 	> [32-bit Git for Windows Setup](https://github.com/git-for-windows/git/releases/download/v2.7.0.windows.1/Git-2.7.0-32-bit.exe)
 	> [64-bit Git for Windows Setup](https://github.com/git-for-windows/git/releases/download/v2.7.0.windows.1/Git-2.7.0-64-bit.exe)

3. git的界面操作工具
	客户端操作工具也是一堆一堆的，本文也不做各种客户端优缺点分析，建议大家下载完美支持gitflow的客户端sourceTree。

	下面是sourceeTree的下载官网，先注册一个账号就可以永久免费使用
	> [sourceTree](https://www.sourcetreeapp.com/)


  以上操作完成之后，先进行初步配置，如下：
```shell
	$ git config --global user.name "Your Name"
	$ git config --global user.email "email@example.com"
```


___

## 添加gitlab账户的ssh访问权限

1. 生成一对ssh公钥私钥，如果.ssh目录下已存在，则跳过此步骤
![](/images/blog/gitlab/ssh_1.png)

2. 查看生成的公钥私钥
![](/images/blog/gitlab/ssh_2.png)

3. 复制ssh公钥，如图
![](/images/blog/gitlab/ssh_3.png)
使用windows的同学，用这个`clip < ~/.ssh/id_rsa.pub`,将ssh公钥复制

以上步骤完成之后，打开并登陆gitlab网站，公司配置的gitlab地址是`192.168.5.227`，ssh端口是`9830`,

gitlab上添加账号的具体操作步骤如下：
![](/images/blog/gitlab/1.png)
![](/images/blog/gitlab/2.png)
![](/images/blog/gitlab/3.png)

以上所有完成之后，可以测试连通性
如果出现以下消息，则代表配置成功了
![](/images/blog/git_connect.png)

---

## 将已有工程托管到gitlab

1. 进入我的工程文件夹路径，比如存放工程的文件夹名称是jiuxian_iOS
![](/images/blog/gitlab/git_1.png)

2. 初始化该工程为git仓库
![](/images/blog/gitlab/git_2.png)

3. 在gitlab创建新的项目
![](/images/blog/gitlab/git_3.png)
![](/images/blog/gitlab/git_4.png)
![](/images/blog/gitlab/git_5.png)
![](/images/blog/gitlab/git_6.png)

4.最后将创建好的项目地址设置为jiuxian_iOS的远端仓库地址
![](/images/blog/gitlab/git_7.png)
做一些说明，gitlab上新建的仓库地址为 `git@xxx.com:ddapps/jiuxian_ios.git` 
如果gitlab服务器端口地址是默认的22，则可以简写为
```
git remote add origin git@xxx.com:ddapps/jiuxian_ios.git
```
因为公司gitlab服务器端口地址修改为9830，需要写成
```
git remote add origin ssh://git@192.168.5.227:9830/ddapps/jiuxian_ios.git
```

接下来是将本地项目托管到gitlab服务器上，
![](/images/blog/gitlab/git_8.png)

到此，gitlab仓库配置就全部完成了。


___

## 将远端仓库的代码克隆到本地

1. 克隆远端仓库代码到我的电脑
![](/images/blog/gitlab/git_8.png)

2. 然后将下载的工程文件夹直接拖入到sourceTree
![](/images/blog/gitlab/git_10.png)
![](/images/blog/gitlab/git_11.png)
![](/images/blog/gitlab/git_12.png)

将远端仓库的代码下载到本地电脑就完成了,下面接着说代码修改和提交的流程。

___

## gitflow流程

1. gitflow初始化
![](/images/blog/gitlab/git_13.png)
![](/images/blog/gitlab/git_14.png)

2. 下面的步骤是需要我完成用户登录的功能，我首先创建新的功能分支 feature/userLogin ,我所有的改动都需要提交，提交之后需要推送到远端push。当我完成这个功能，我就可以finish这个分支。
![](/images/blog/gitlab/git_15.png)
![](/images/blog/gitlab/git_16.png)
![](/images/blog/gitlab/git_17.png)
![](/images/blog/gitlab/git_18.png)
![](/images/blog/gitlab/git_19.png)
![](/images/blog/gitlab/git_20.png)

___

Merge Request
![](/images/blog/gitlab/merge_1.png)
![](/images/blog/gitlab/merge_2.png)
![](/images/blog/gitlab/merge_3.png)
![](/images/blog/gitlab/merge_4.png)
![](/images/blog/gitlab/merge_5.png)



___

以上是整个基本的操作流程，下面介绍基本的命令行操作

```
git add 
git commit 
git fetch 
git pull 
git merge
git branch
git reset
git rebase
```






