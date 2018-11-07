---
title: GITHUB：协同开发
date: 2018-06-04 13:36:43
tags: [github,git]
categories: github
---

# 如何将项目代码上传到github管理

## 下载git

官网 [链接](https://git-scm.com/downloads)

下载完成一直next即可。

任何文件夹下，点鼠标右键会多出一些菜单
如 Git Init Hear、Git Bash、Git Gui ， 说明安装成功。

## 注册github账户

官网注册 [官网](https://github.com/)

<!-- more -->

## 创建github仓库

页面上方用户菜单上选择 “+”->New repository 创建一个新的仓库来协同开发

![github](/images/2018-06-04/github_1.png)

## 创建本地仓库

1. 在本地需要上传项目的目录下鼠标右键，选择：Git Bash Here，执行命令 git init
	
	```bash
	git init
	```

	> Git 使用 git init 命令来初始化一个 Git 仓库，Git 的很多命令都需要在 Git 的仓库中运行，所以 git init 是使用 Git 的第一个命令。在执行完成 git init 命令后，Git 仓库会生成一个 .git 目录，该目录包含了资源的所有元数据，其他的项目目录保持不变（不像 SVN 会在每个子目录生成 .svn 目录，Git 只在仓库的根目录生成 .git 目录）。

	输入指令之后，回车，你就会看到的项目路径下会多出了一个.git文件夹

2. 创建 SSH key

	众所周知ssh是加密传输。加密传输的算法有好多，git可使用rsa，rsa要解决的一个核心问题是，如何使用一对特定的数字，使其中一个数字可以用来加密，而另外一个数字可以用来解密。这两个数字就是你在使用git和github的时候所遇到的public key也就是公钥以及privatekey私钥。其中，公钥就是那个用来加密的数字，这也就是为什么你在本机生成了公钥之后，要上传到github的原因。从github发回来的，用那公钥加密过的数据，可以用你本地的私钥来还原。如果你的key丢失了，不管是公钥还是私钥，丢失一个都不能用了，解决方法也很简单，重新再生成一次，然后在github.com里再设置一次就行。

	```bash
	ssh-keygen -t rsa -C "youremail@example.com" //youremail@example.com是在github注册时候使用的邮箱
	```

	出现的步骤一直回车即可（如下图：）
	![github-ssh](/images/2018-05-31/hexo_github_ssh.png "SSH")

	这样在 /c/Users/用户名/.ssh/id_rsa文件中就生成了公钥

3. 配置github账户的ssh key

	拷贝 /c/Users/用户名/.ssh/id_rsa 的公钥内容，进入你的github设置Settings
	![github-ssh](/images/2018-05-31/hexo_github_ssh_1.png "SSH")

	将公钥贴到Key中，title可以为空
	![github-ssh](/images/2018-05-31/hexo_github_ssh_2.png "SSH")

	添加好之后验证

	```bash
	$ ssh -T git@github.com
	```

	提示信息为下面语句说明key已经配置好了

	```bash
	Hi github_user_name! You've successfully authenticated, but GitHub does not provide shell access.
	```

## 项目上传github

1. 把本地仓库传到github上去，在此之前还需要设置username和email，因为github每次commit都会记录他们

	```bash
	$ git config --global user.name "姓名"
	$ git config --global user.email "邮箱"
	```

2. 进入要上传的仓库，右键git bash，添加远程地址
	```bash
	git remote add origin git@github.com:yourName/yourRepository.git
	```

	后面的yourName和yourRepo表示你再github的用户名和刚才新建的仓库，加完之后进入.git，打开config，这里会多出一个remote “origin”内容，这就是刚才添加的远程地址，也可以直接修改config来配置远程地址。

3. github上传，更新项目

	```bash
	git add -A  // 也可以单独提交文件， -A改成文件即可
	git commit -m "test commit" // -m 添加提交注释
	git push origin master
	```

## gitignore文件

.gitignore顾名思义就是告诉git需要忽略的文件，这是一个很重要并且很实用的文件。一般我们写完代码后会执行编译、调试等操作，这期间会产生很多中间文件和可执行文件，这些都不是代码文件，是不需要git来管理的。我们在git status的时候会看到很多这样的文件，如果用git add -A来添加的话会把他们都加进去，而手动一个个添加的话也太麻烦了。这时我们就需要.gitignore了。

# github上协作开发

## 添加github账户

进入项目的仓库，Settings —》 Collaborators 添加github账户，如下图

![github](/images/2018-06-04/github_2.png "SSH")

会向上图填写的github账户绑定的邮箱发送邀请的邮件，点击链接跳转网页，点击accept invitation就可以了。

同样配置SSH Key（配置方法见上面）

下载项目
```bash
git clone 项目路径
```

更改文件提交先更新最新代码

```bash
git pull
git add 更改的文件
git commit -m "update test"
git push
```

回到Leader的账号，可以看到刚才提交的东西已经同步到了该远程仓库 


> 参考：http://blog.csdn.net/gpwner/article/details/52829187