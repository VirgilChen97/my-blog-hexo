---
title: SpringBoot 日志
date: 2020-06-13 13:03:37
tags: Spring, SpringBoot
categories: 找工作
---

在写这篇笔记之前我也有很多思考，我参考的教程中有很多SpringMVC以及JSP，模板引擎的内容。貌似现在的开发大环境下，后端需要兼顾手机App和网页版共同的请求，前后端分离是大的趋势。但是我仍然准备学习一下传统的服务器端渲染的相关技术。一方面是感受一下技术的演进，另一方面是前后端分离不利于SEO（Serach Engine Optimization），很多现有的产品仍然使用服务器渲染的原因就是搜索引擎。虽然现在可能会有一些更加成熟的方案（nodejs作为中间层），但是咱们还是一步一步来。

<!--more-->

# Spring Boot 静态资源映射

在以往的SpringMVC中，我们会将静态资源文件储存在WEBAPP中，但是在SpringBoot中则有所不同。

## 1. `/webjars/**`: jar包形式存在的静态资源

所有对于 `/webjars/**` 的访问都会去 `classpath:/META-INF/resources/webjars/` 下去找相应的资源。你可以通过 [webjars](https://www.webjars.org/) 网站获取相应webjar的maven依赖，例如 `jQuery` 的依赖如下：

```xml
<!--引入Jquery-->
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.5.1</version>
</dependency>
```

你同样可以找到其他流行的web框架例如 Bootstrap, react 的webjar。作为测试我们吧 `jQuery` 的依赖添加到我们的项目中。在Idea的External Libraries中我们可以看到 `jQuery` 的 webjar 结构如下：

![](img/2020-06-14-14-23-28.png)

所以如果此时我们启动项目，直接在浏览器访问 `localhost:8080/webjars/jquery/3.5.1/jquery.js` 我们便可以获取 `jquery.js` 这个文件的内容：

![](img/2020-06-14-14-28-09.png)

## 2. `/**` 静态资源文件夹

所有对于 `/**` 路径的访问，如果没有被 `@RequestMapping` 绑定，那么SpringBoot会默认从一下文件夹获取：

```java
"classpath:/META-INF/resources"
"classpath:/resources"  //注意这个不是我们创建工程中已经存在的resources，工程中已存在的resource实际是根目录
"classpath:/static"
"classpath:/public"
"/"
```

例如我们现在在 `resource/static` 文件夹下创建一个test.html:

![](img/2020-06-14-14-54-59.png)

内容为：

```html
<h1>Hello</h1>
```

此时我们访问 `localhost:8080/test.html` ：

![](img/2020-06-14-14-56-21.png)

可以看到资源成功访问，载入了test.html页面。

## 3. 欢迎页映射：index.html

如果我们直接访问 `localhost:8080/`，那么 SpringBoot 就会去每个资源文件夹下寻找index.html并返回，把刚才我们创建的 HTML 文件改名为 `index.html`，在浏览器中访问 `localhost:8080/` ，可以发现页面依然成功载入了。

## 4. favicon

定义网页图标，所有静态资源文件夹下的favicon.ico

# SpringBoot 模板引擎

JSP, velocity, Thymeleaf都是模板引擎。他们可以讲模板与数据结合，并生成最终的页面。SpringBoot 推荐的模板引擎时Thymeleaf。修改 pom.xml 引入 thymeleaf：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

## Demo

Thymeleaf 使用起来十分简单，我们只需要吧作为模板的HTML文件放在 `templates` 文件夹下，thymeleaf就会自动为我们渲染页面。我们先从一个简单的例子出发，首先在 `controller` 包下新建一个 `HelloController` 类，内容如下：

```java
@Controller // 不是RestContoller，因为我们返回的不再是responsebody
public class HelloController {
    @RequestMapping("/success") // 处理 /success请求
    public String success(){
        return "success"; // 返回模板页面名称
    }
}
```

然后我们在 `resource/template` 文件夹下创建一个 `success.html`，内容如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Success!</title>
</head>
<body>
<h1>This is a success message</h1>
</body>
</html>
```

运行项目，访问 `localhost:8080/success`:

![](img/2020-06-14-15-23-50.png)

更加具体的语法可以参考 [Thymeleaf 官方文档](https://www.thymeleaf.org/documentation.html)









