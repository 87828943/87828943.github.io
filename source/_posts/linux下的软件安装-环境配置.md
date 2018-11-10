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

# mysql安装
1. mysql官网下载[官网下载](https://www.mysql.com/downloads/)

2. 卸载旧mysql并安装
	如果有旧版本mysql删除：
	```bash
	find / -name mysql
	rm -rf #上边查找到的路径
	```

	安装：
	```bash
	tar -zxvf mysql-5.6.42-linux-glibc2.12-x86_64.tar.gz #解压
	rm -rf mysql-5.6.42-linux-glibc2.12-x86_64.tar.gz #删除安装包
	mv mysql-5.6.42-linux-glibc2.12-x86_64/ mysql #更名mysql
	```

3. 添加mysql用户组和mysql用户
	先检查是否有mysql用户组和mysql用户
	```bash
	groups mysql
	```

	若无，则添加
	```bash
	groupadd mysql
	useradd -r -g mysql mysql
	```

4. 进入mysql目录更改权限

	```bash
	cd mysql/
	chown -R mysql:mysql ./ #修改目录拥有者为mysql用户
	```

5. 执行安装脚本

	```bash
	./scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
	```

	5.7.6之后的版本初始化数据库不再使用mysql_install_db，而是使用： bin/mysqld --initialize

	***出现以下异常信息***：
	<font color="red">FATAL ERROR: please install the following Perl modules before executing scripts/mysql_install_db:Data::Dumper</font>

	```bash
	#解决方法（缺少安装包perl-Data-Dumper）：
	yum install -y perl-Data-Dumper
	```
	<font color="red">error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory</font>
	```bash
	#解决方法（缺少安装包libaio和libaio-devel.）：
	yum install libaio*
	```

	安装完之后修改当前目录拥有者为root用户，修改data目录拥有者为mysql

	```bash
	chown -R root:root ./ #修改当前目录拥有者为root用户
	chown -R mysql:mysql data #修改当前data目录拥有者为mysql用户
	```

6. 添加mysql自启动
	
	```bash
	cp support-files/mysql.server /etc/init.d/mysql
	chkconfig --add mysql #添加系统服务
	chkconfig mysql on
	#创建缺少的文件
	mkdir /var/log/mariadb
	touch /var/log/mariadb/mariadb.log
	#添加mysql命令
	ln -s /usr/local/mysql/bin/mysql /usr/bin
	service mysql start #启动mysql服务
	```

7. 更改mysql密码

	```bash
	./bin/mysqladmin -u root password 'root' #更改密码
	```
	***更改密码出现以下异常信息***：
	<font color="red">Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)</font>

	```bash
	#解决方法：打开/etc/my.cnf,看看里面配置的socket位置是什么目录。
	#“socket=/var/lib/mysql/mysql.sock”路径和“/tmp/mysql.sock”不一致。建立一个软连接：
	ln -s /var/lib/mysql/mysql.sock /tmp/mysql.sock
	```

8. 增加远程登录权限

	本地登陆MySQL后执行如下命令
	```bash
	grant all privileges on *.* to root@'%' identified by 'root';
	flush privileges;
	```
	* grant all privileges on 库名.表名 to '用户名'@'IP地址' identified by '密码' with grant option;
	* 库名:要远程访问的数据库名称,所有的数据库使用“*” 
	* 表名:要远程访问的数据库下的表的名称，所有的表使用“*” 
	* 用户名:要赋给远程访问权限的用户名称 
	* IP地址:可以远程访问的电脑的IP地址，所有的地址使用“%” 
	* 密码:要赋给远程访问权限的用户对应使用的密码

9. 配置my.cnf

	参考：[my.cnf配置参数](http://www.cnblogs.com/lyq863987322/p/8074749.html)
	```bash
	vim my.cnf
	#添加以下语句并保存退出
	lower_case_table_names=1
	max_allowed_packet=100M
	#配置好之后，重启mysqld服务
	service mysql restart
	```

# jenkins安装

> 前置准备java环境，安装JDK，安装方式：rpm包下载安装

1. 官网下载[官网下载](https://jenkins.io/download/)

2. 解压安装

	```bash
	rpm -ih jenkins-1.562-1.1.noarch.rpm
	rm -rf jenkins-1.562-1.1.noarch.rpm
	```

	自动安装完成之后： 

	* /usr/lib/jenkins/jenkins.war    WAR包 
	* /etc/sysconfig/jenkins       配置文件
	* /var/lib/jenkins/        默认的JENKINS_HOME目录
	* /var/log/jenkins/jenkins.log    Jenkins日志文件

3. 配置jenkins

	```bash
	vi /etc/sysconfig/jenkins
	```

	进入jenkins的系统配置文件并修改相关端口号（也可以不修改）
	jenkins的默认JENKINS_PORT是8080，JENKINS_AJP_PORT默认端口是8009，这同tomcat的默认端口冲突。我这更改为8088和8089。

	```bash
	vi /etc/init.d/jenkins
	```

	加入java环境变量
	candidates="/usr/java/jdk1.8.0_191/bin/java"

4. 启动
	```bash
	service jenkins start
	```

5. jenkins服务安装

	提示安装自定义插件还是推荐插件，选择推荐插件：
	![](/images/2018-11-08/jenkins_1.png)
	![](/images/2018-11-08/jenkins_2.png)
	创建管理员用户
	![](/images/2018-11-08/jenkins_3.png)
	![](/images/2018-11-08/jenkins_4.png)

# git安装

1. 下载
	获取github最新的Git安装包下载链接并解压
	```bash
	wget https://github.com/git/git/archive/v2.17.0.tar.gz
	tar -zxvf v2.17.0.tar.gz
	rm -rf v2.17.0.tar.gz
	```

2. 安装编译源码所需依赖

	```bash
	yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
	```
	耐心等待安装，出现提示输入y即可；

	安装依赖时，yum自动安装了Git，需要卸载旧版本Git出现提示输入y即可；
	```bash
	yum remove git
	```
	出现提示输入y即可

3. 编译安装

	```bash
	cd git-2.17.0
	make prefix=/usr/local/git all
	#安装Git至/usr/local/git路径
	make prefix=/usr/local/git install
	```

4. 配置环境变量 
	打开环境变量配置文件，
	```bash
	vim /etc/profile
	```
	在底部加上Git相关配置信息
	***
	PATH=$PATH:/usr/local/git/bin
	export PATH
	***
	```bash
	#让更改立刻生效
	source /etc/profile
	```
5. 验证

	输入命令 git version ，查看安装的git版本，安装成功。