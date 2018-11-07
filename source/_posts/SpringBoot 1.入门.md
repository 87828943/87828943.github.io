---
title: SpringBoot 1.入门
date: 2018-06-01 11:03:05
tags: [springboot]
categories: springboot
---

# springboot

## 介绍

Spring Boot是由Pivotal团队提供的全新框架，是为了简化Spring应用的创建、运行、调试、部署等而出现的，“约定大于配置”，其实springboot就是默认配置了平常构建spring项目需要的配置文件，使开发人员无需过多关注许多的xml配置等。

## 好处

1. 独立运行的Spring项目
2. 内嵌Servlet容器
3. 提供starter简化Maven配置
4. 自动配置Spring 
5. 无代码生成和xml配置

<!-- more -->

更详细的优缺点就不一一罗列了

** 总而言之，只需要非常少的配置文件就可以快速搭建起来一个web项目或者分布式微服务 **

## 搭建项目（列举下面两种）

### ①可以通过http://start.spring.io/ 来快速搭建项目

1. 进入官网 http://start.spring.io/
![springboot](/images/2018-06-01/springboot_1_1.png)

2. 填写项目信息以及自定义配置，点击 Switch to the full version. ，可以添加更多需要使用的技术（Web，SQL，NoSQL，Template Engines等）

3. 点击generate project下载项目压缩包

4. 解压到工作空间，用编程工具eclipse，idea导入maven项目即可


### ②通过idea或者eclipse直接搭建项目

我使用的是idea，通过idea构建springboot项目

1. 首先新建项目 File -> new 如下图，选择Spring Initializr ，url填 http://start.spring.io/ 然后Next
 ![springboot](/images/2018-06-01/springboot_1_2.png)
2. 填写项目信息，Next
![springboot](/images/2018-06-01/springboot_1_3.png)
3. 选择需要用到的技术（勾选web说明创建一个web项目，注：springboot默认集成tomcat等容器），Next
![springboot](/images/2018-06-01/springboot_1_4.png)
4. 项目名，项目路径相关，Finish！
![springboot](/images/2018-06-01/springboot_1_5.png)

## 项目结构

![springboot](/images/2018-06-01/springboot_1_6.png)

如图，基础结构就三个部分

* src/main/java 程序开发以及主程序入口
* src/main/resources 配置文件
* src/test/java 测试程序

pom.xml文件中默认有两个模块：

1. spring-boot-starter ：核心模块，包括自动配置支持、日志和YAML；
2. spring-boot-starter-test ：测试模块，包括JUnit、Hamcrest、Mockito。

## 运行

1. 上面构建项目的时候已经勾选了web，构建好就是一个web项目，其实就是在pom中默认了下面的依赖
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
2. 编写Hello World
```java
@RestController
public class HelloWorldController {

    @RequestMapping("/helloWorld")
    public String index(){
        return "Hello World";
    }
}

```
    @RestController其实就是@ResponseBody跟@Controller合在一起，默认返回json格式对象，不用添加jackjson配置之类。

3. 启动DemoApplication，访问http://localhost:8080/helloWorld 就可以看到结果了。

![springboot](/images/2018-06-01/springboot_1_7.png)
![springboot](/images/2018-06-01/springboot_1_8.png)

## 其他

### 测试

示例：使用mockmvc进行，利用MockMvcResultHandlers.print()打印出执行结果。

![springboot](/images/2018-06-01/springboot_1_9.png)

注： MockMvcBuilders.standaloneSetup(Object... controllers)：通过参数指定一组控制器，这样就不需要从上下文获取了；

### 热部署

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
            </configuration>
        </plugin>
	</plugins>
</build>
```
解决 IntelliJ IDEA 无法热加载 Spring Boot 静态资源文件:
参考： http://fanlychie.github.io/post/resolve-intellij-idea-spring-boot-template-reload-is-not-working.html

1. 快捷键Ctrl + Alt + S打开设置面板，勾选Build project automatically选项：
    ![热加载静态资源文件](/images/2018-06-01/idea-settings.png)
2. 快捷键Ctrl + Shift + A查找registry命令：
    ![热加载静态资源文件](/images/2018-06-01/registry.png)
3. 在查找到的registry命令通过鼠标双击或敲回车键，在弹出的面板中搜索关键字automake，找到并勾选compiler.automake.allow.when.app.running选项：
    ![热加载静态资源文件](/images/2018-06-01/registry-automake.png)

### 资源文件

application.properties

```
mars.md5.password.key=marsXXX
mars.session.user.key=mars_user_key
default.logo=/img/testlogo.png
```

java使用

```java
@Value("${mars.md5.password.key}")
private String MARS_MD5_PASSWORD_KEY;
@Value("${mars.session.user.key}")
private String MARS_SESSION_USER_KEY;
@Value("${default.logo}")
private String DEFAULT_LOGO;
```

## 结论

spring boot可以非常方便、快速搭建项目，开发人员不用关心框架之间的兼容性，适用版本等问题，想使用任何东西，仅仅添加一个配置就可以，所以sping boot非常适合构建微服务。