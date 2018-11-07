---
title: 通过HEXO部署GITHUB搭建个人博客
date: 2018-05-31 16:13:11
tags: [hexo]
---

> 自行注册github，创建存放静态文件的仓库，仓库地址即下面_config.yml的repository（此处略过）

## hexo配置

自动部署首先需要安装hexo-deployer-git

```bash
$ npm install hexo-deployer-git --save
```

然后配置_config.yml中的deploy

```bash
deploy:
 type: git
 repository:git@github.com:github_user_name/github_user_name.github.io.git
 branch: master
 ```

 ** 注意：这里的repo需要设置成你git仓库的ssh链接 **

 <!-- more -->

## 生成 ssh key

命令行中输入：

```bash
$ ssh-keygen -t rsa -C 此处换成你的邮箱地址
```

出现的步骤一直回车即可（如下图：）
![github-ssh](/images/2018-05-31/hexo_github_ssh.png "SSH")

这样在 /c/Users/用户名/.ssh/id_rsa文件中就生成了公钥

## 配置github账户的ssh key

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

## 初始化本地git仓库

```bash
$ git config --global user.name "姓名"
$ git config --global user.email "邮箱"
```

在本地的hexo init生成的文件夹中初始化git仓库：

```bash
$ git init
```

** OK,现在通过 hexo d 已经可以不输入密码部署到guthub了！ **

## 通过域名访问github地址(略过)