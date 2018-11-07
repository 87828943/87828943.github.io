---
title: linux下的软件安装&环境配置
date: 2018-11-07 16:08:30
tags: [linux]
---



【存档备份】

<!-- more -->

# linux与Window之间的文件传输

## rz sz 上传下载

安装rz，sz，操作很简单。

```bash
yum install lrzsz
```

安装之后，就可以进行基本的上传下载操作：

通过SecureCRT执行rz，进行上传操作。

```bash
[root@iZ25ltqcjzrZ ~]# rz （上传，会弹出窗口，选择上传文件，等待完成。）
[root@iZ25ltqcjzrZ ~]# sz 文件名（下载）
```

## scp传输文件

由于部分服务器安全考虑，不让安装rz软件，只好使用scp传输。scp的操作命令其实很简单。
将本地文件传输的到目标服务器的指定路径下：

```bash
# 文件复制
$scp local_file remote_username@remote_ip:remote_folder
# 目录复制
$scp -r local_folder remote_username@remote_ip:remote_folder
```
其中local_file为本地文件，remote_username目标服务器登录名称，remote_ip目标服务器密码，remote_folder目标服务器下的目标路径。

将远程文件cp到本地：

```bash
$scp remote_username@remote_ip:remote_file local_folder
```

## SFTP

SecureCRT可以通过快捷键Alt+p进入sftp连接模式。

下载文件

```bash
sftp>get 文件绝对路径
```

查看下载到本地的路径，得到下载到本地的路径

```bash
sftp>lpwd
```

上传文件：

```bash
sftp>put 本地文件绝对路径
```


# jdk安装

> 由于使用 yum 或者 apt-get 命令 安装 openjdk 可能存在类库不全，从而导致用户在安装后运行相关工具时可能报错的问题，推荐采用手动解压安装的方式来安装 JDK。

1. oracle官网下载jdk[官网下载](https://www.oracle.com/technetwork/java/javase/overview/index.html)
2. 创建目录
	在/usr/目录下创建java目录
	```bash
		mkdir /usr/java
		cd /usr/java
	```
	把下载的文件 jdk-8u151-linux-x64.tar.gz 复制到在/usr/java/目录下。
3. 解压jdk
	```bash
	tar -zxvf jdk-8u151-linux-x64.tar.gz
	```
4. 设置环境变量
	```bash
	vi /etc/profile
	```
	在 profile 文件中添加如下内容并保存：
	```bash
	JAVA_HOME=/usr/java/jdk1.8.0_191        
	JRE_HOME=/usr/java/jdk1.8.0_191/jre     
	CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
	PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
	export JAVA_HOME JRE_HOME CLASS_PATH PATH
	```
	注意：其中 JAVA_HOME， JRE_HOME 请根据自己的实际安装路径及 JDK 版本配置。

	让修改生效(或者直接重启)：
	```bash
	source /etc/profile
	```
5. 验证
	```bash
	java -version
	```

	结果：
	
	```bash
	java version "1.8.0_191"
	Java(TM) SE Runtime Environment (build 1.8.0_191-b12)
	Java HotSpot(TM) 64-Bit Server VM (build 25.191-b12, mixed mode)
	```

# redis安装

# jenkins安装

