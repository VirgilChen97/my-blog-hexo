---
title: SpringBoot-Web开发（三）
date: 2020-06-22 23:32:09
tags: 
    - Spring
    - SpringBoot
categories: 找工作
toc: true
---

第三部分我们学习Spring Boot的错误处理机制，以及我们如何自定义错误数据

# 错误处理

为了方便测试，我们先关闭拦截器，注释掉相应代码。如果此时访问一个项目中不存在的地址，我们会得到一个默认的错误页面：

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-19-19-44-54.png)

<!-- more -->

如果此时我们使用Postman向服务器像一个不存在的地址发送请求，我们会得到一个json数据：

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-19-19-47-18.png)

SpringBoot 会根据请求头中的 `Accept:` 属性来决定做出哪种请求相应。SpringBoot是如何实现的呢？我们查看 `ErrorMvcAutoConfiguration` 自动配置类发现，SpringBoot自动为我们配置了以下几个组件：

- `ErrorPageCustomizer`：当服务器发生错误时，重定向到错误请求 (`/error`)
- `BasicErrorController`：处理错误请求的Controller，读取请求头的 `Accept:` 决定调用的方法类型
- `DefaultErrorViewResolver`： `BasicErrorController` 会通过 ErrorViewResolver接口来找到需要返回的视图，`DefaultErrorViewResolver` 就是Spring默认的实现。该实现会首先尝试模板引擎是否有适用于错误的 View，如果没有则会去静态资源目录找静态资源，如果都没有则返回null
- `DefaultErrorAttributes`：当使用模板引擎的时候，我们可以获取错误相关的参数，而 `ErrorAttributes` 接口则定义了获取错误参数的行为，`DefaultErrorAttributes` 就是SpringBoot的默认是实现，其中我们可以获取到的参数有：
    - `timestamp`：时间戳
    - `status`：状态码
    - `error`：错误提示
    - `exception`：异常对象
    - `message`：异常消息
    - `errors`：JSR303数据校验错误

对原理有了一定了解后，我们就知道如何定制我们的错误页面了。对于定制 HTML 错误页面，我们只需要讲错误页面的模板放在模板文件的 `error/` 目录下，就会自动调用。例如如果发生了404错误，那么Spring就会去找 `error/404.html` 这个模板，对于其他错误，Spring都会寻找 `error/{错误状态码}.html`。你也可以通过 `4xx.html` 来响应所有的 4xx 错误。现在尝试在 `templates` 文件夹中创建 `error` 文件夹，把 404.html 移动到 `error`，修改404.html，尝试取出错误参数：

```html
<main role="main" class="col-md-9 ml-sm-auto col-lg-10 pt-3 px-4">
    <h1>status: [[${status}]]</h1>
    <h2>Timestamp: [[${timestamp}]]</h2>
    <p>Error: [[${error}]]</p>
</main>
```

此时再次访问不存在的地址：

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-19-20-26-13.png)

已经变成了我们定制的错误页面。现在我们完成了错误页面的定制，那么我们如何实现错误数据的定制呢？首先我们需要为指定的错误创建对应的Exception。假设我们现在有一个请求是查找员工，地址是 `/employee/{id}`，当查询一个不存在的员工时，我们希望返回 404 错误，并且提示 “用户未找到”。那么首先我们创建一个Exception包，在里面创建 `EmployeeNotFoundException` 类：



我们之前提到的组件 `DefaultErrorAttributes` 包含了错误数据的格式，那么我们自己也可以实现一个 `ErrorAttributes` 并加入容器来实现错误数据的定制。在 `component` 包下创建

