---
title: SpringBoot 入门
date: 2020-06-12 16:48:38
tags: Spring, SpringBoot
categories: 找工作
---

# Hello World

## 1. 使用IDEA中的Spring Initializer创建一个新工程

初始项目中选择Spring Web，创建好以后目录结构如下

```
.
├── HELP.md
├── demo.iml
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── cyf
    │   │           └── demo
    │   │               └── DemoApplication.java
    │   └── resources
    │       ├── application.properties
    │       ├── static
    │       └── templates
    └── test
        └── java
            └── com
                └── cyf
                    └── demo
                        └── DemoApplicationTests.java
```

<!--more-->

## 2. 编写一个新的controller

我们在 `com.cyf.demo` 下创建一个新的 `controller` 包。新建 `HelloController` 类。内容如下：

```java
package com.cyf.demo.contrtoller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController //是 @Controller 和 @ResponseBody 的结合
public class HelloController {
    @RequestMapping("/hello") // 指定访问 /hello 时调用此函数
    public String hello(){
        return "Hello World!";
    }
}
```

## 3. 运行项目

回到我们的 `DemoApplication`, 点击旁边的绿色小三角：

![](/img/2020-06-12-17-28-38.png)

可以看到Tomcat自动启动，服务已经跑起来了

![](img/2020-06-12-17-29-30.png)

此时在浏览器访问 `localhost:8080/hello`

![](img/2020-06-12-17-30-34.png)

可以看到返回了我们在新建的Controller中定义的字符串