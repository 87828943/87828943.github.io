---
title: HEXO & MELODY
date: 2018-05-30 16:13:11
tags: [hexo]
categories: hexo
---
> Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

More info: [Hexo](https://hexo.io/zh-cn/)

## 安裝

安装前，必须安装下列应用程序

1. node.js [官网下载](https://nodejs.org/en/)
2. git [官网下载](https://git-scm.com/)

安装完成后只需要使用npm即可安装

```
$ npm install -g hexo-cli
```

<!-- more -->

## 建站

### 初始化

安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。

``` bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```

新建完成后，指定文件夹的目录如下：

```
.
├── .deploy
├── public
├── scaffolds
├── scripts
├── source
|   ├── _drafts
|   └── _posts
├── themes
├── _config.yml
└── package.json
```

* .deploy：执行hexo deploy命令部署到GitHub上的内容目录
* public：执行hexo generate命令，输出的静态网页内容目录
* scaffolds：layout模板文件目录，其中的md文件可以添加编辑
* scripts：扩展脚本目录，这里可以自定义一些javascript脚本
* source：文章源码目录，该目录下的markdown和html文件均会被hexo处理。该页面对应repo的根目录，404文件、favicon.ico文件，CNAME文件等都应该放这里，该目录下可新建页面目录。
	* _drafts：草稿文章
	* _posts：发布文章
* themes：主题文件目录
* _config.yml：全局配置文件，大多数的设置都在这里
* package.json：应用程序数据，指明hexo的版本等信息，类似于一般软件中的 关于 按钮

### 命令

仅列举常用命令

``` bash
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy #将.deploy目录部署到GitHub
```

常用复合命令

``` bash
hexo deploy -g
hexo server -g
```

简写：

``` bash
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy
```

## 主题

本项目应用主题：melody  [链接](https://github.com/Molunerfinn/hexo-theme-melody)

melody操作文档  [链接](https://molunerfinn.com/hexo-theme-melody-doc/#/)


安装

``` bash
$ git clone -b master https://github.com/Molunerfinn/hexo-theme-melody themes/melody
```

If you don't have jade & stylus renderer, follow this:

``` bash
$ npm install hexo-renderer-jade hexo-renderer-stylus
```

背景线粒效果
["点击这里下载：canvas-nest.js-master.zip"](/files/canvas-nest/canvas-nest.js-master.zip)

## 工作空间迁移（换电脑）

1. 可以通过github维护一个新的分支，一个存静态文件用于展示，一个存放源生文件。
2. 新电脑搭建好环境后，只需要复制以下文件或目录进行覆盖即可。
	* _config.yml  	（包含个性化个人站点配置）
	* package.json 	（当前版本信息）
	* scaffolds/ 	（存放个性化模板，没改动过的话不需要）
	* source/ 		（存放文章以及文件）
	* themes/		（个性化主题）
