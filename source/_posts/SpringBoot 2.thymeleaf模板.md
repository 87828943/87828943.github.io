---
title: SpringBoot 2.thymeleaf模板
date: 2018-06-04 14:43:44
tags: [springboot,thymeleaf]
categories: springboot
---

# thymeleaf的使用

## thymeleaf介绍

Springboot默认是不支持JSP的，默认使用thymeleaf模板引擎。

> Thymeleaf是一款用于渲染XML/XHTML/HTML5内容的模板引擎。类似JSP，Velocity，FreeMaker等，它也可以轻易的与Spring MVC等Web框架进行集成作为Web应用的模板引擎。与其它模板引擎相比，Thymeleaf最大的特点是能够直接在浏览器中打开并正确显示模板页面，而不需要启动整个Web应用。

<!-- more -->

## 配置

1. 在application.properties文件中增加Thymeleaf模板的配置。

	```
	#thymelea模板配置
	spring.thymeleaf.prefix=classpath:/templates/
	spring.thymeleaf.suffix=.html
	spring.thymeleaf.mode=HTML5
	spring.thymeleaf.encoding=UTF-8
	spring.thymeleaf.content-type=text/html
	spring.thymeleaf.cache=false //开发时关闭缓存,不然没法看到实时页面
	spring.resources.chain.strategy.content.enabled=true
	spring.resources.chain.strategy.content.paths=/**
	```
	这些配置不是必须的，如果配置了会覆盖默认的。

2. pom.xml添加依赖

	```
	<dependency>
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter-thymeleaf</artifactId>
	</dependency>
	```

3. 目录结构
	* spring-boot项目静态文件目录：/src/java/resources/static 
	* spring-boot项目模板文件目录：/src/java/resources/templates 
	* spring-boot静态首页的支持，即index.html放在以下目录结构会直接映射到应用的根目录下

## 测试使用

编写一个demo Controller

```java
@RequestMapping(value = "/",method = RequestMethod.GET)
private String index(Model model){
    model.addAttribute("title","thymeleaf测试");
    return "index";
}
```

写index.html页面

```XML
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Index!</title>
</head>
<body>
<p th:text="${title}"/>
</body>
</html>
```

启动springboot 访问 http://localhost:8080 如下图

![thymeleaf](/images/2018-06-04/springboot_2_1.png)

## thymeleaf常用基础知识点

1. 在html页面中引入thymeleaf命名空间，即<html xmlns:th=http://www.thymeleaf.org></html>，此时在html模板文件中动态的属性使用th:命名空间修饰 

2. 引用静态资源文件，比如CSS和JS文件，语法格式为“@{}”，如@{/js/blog/blog.js}会引入/static目录下的/js/blog/blog.js文件

3. 访问spring-mvc中model的属性，语法格式为“${}”，如${user.id}可以获取model里的user对象的id属性 

4. 循环，在html的标签中，加入th:each=“value:${list}”形式的属性，如<span th:each=”user:${users}”></span>可以迭代users的数据 

5. 判断，在html标签中，加入th:if=”表达式”可以根据条件显示html元素 

	```XML
	<span th:if="${not #lists.isEmpty(blog.publishTime)}"> 
		<span id="publishtime" th:text="${#dates.format(blog.publishTime, 'yyyy-MM-dd HH:mm:ss')}"></span> 
	</span> 
	```
	以上代码表示若blog.publishTime时间不为空，则显示时间

6. 时间的格式化 
	```
	${#dates.format(blog.publishTime,'yyyy-MM-dd HH:mm:ss')}
	```	
7. 符串拼接  th:href="'/blog/delete/' + ${blog.id }"

> 参考：https://blog.csdn.net/u014695188/article/details/52347318

