---
title: SpringBoot Web 开发
date: 2020-06-14 13:03:37
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

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-14-14-23-28.png)

所以如果此时我们启动项目，直接在浏览器访问 `localhost:8080/webjars/jquery/3.5.1/jquery.js` 我们便可以获取 `jquery.js` 这个文件的内容：

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-14-14-28-09.png)

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

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-14-14-54-59.png)

内容为：

```html
<h1>Hello</h1>
```

此时我们访问 `localhost:8080/test.html` ：

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-14-14-56-21.png)

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

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-14-15-23-50.png)

更加具体的语法可以参考 [Thymeleaf 官方文档](https://www.thymeleaf.org/documentation.html)

# CRUD 实例

## 引入静态资源

首先讲我们实验的 [基础代码](https://drive.google.com/file/d/1jtvP4I8anneabwiIyyNy4FTJnVSZm_D3/view?usp=sharing) 复制到项目中。将 `dao` 和 `entities` 复制到项目中，这里我使用 Spring Initializer 新建了一个 restful-api的项目。同时将我们的 html, css, js 文件导入项目。完成后目录的结构如下：

```
src
├── main
│   ├── java
│   │   └── com
│   │       └── cyf
│   │           └── restfulapi
│   │               ├── controller
│   │               │   └── HelloController.java
│   │               ├── dao
│   │               │   ├── DepartmentDao.java
│   │               │   └── EmployeeDao.java
│   │               ├── entities
│   │               │   ├── Department.java
│   │               │   └── Employee.java
│   │               └── RestfulApiApplication.java
│   └── resources
│       ├── application.properties
│       ├── static
│       │   └── asserts
│       │       ├── css
│       │       │   ├── bootstrap.min.css
│       │       │   ├── dashboard.css
│       │       │   └── signin.css
│       │       ├── img
│       │       │   └── bootstrap-solid.svg
│       │       └── js
│       │           ├── bootstrap.min.js
│       │           ├── Chart.min.js
│       │           ├── feather.min.js
│       │           ├── jquery-3.2.1.slim.min.js
│       │           └── popper.min.js
│       └── templates
│           ├── 404.html
│           ├── dashboard.html
│           ├── index.html
│           ├── list.html
│           └── success.html
└── test
    └── java
        └── com
            └── cyf
                └── restfulapi
                    └── RestfulApiApplicationTests.java

```

根据我们之前学习的静态资源映射规则，现在我们启动项目，访问 `lcoalhost:8080`，可以发现首页已经显示出来了：

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-15-12-59-16.png)

目前我们的HTML文件名称就叫做 index.html，那么如果我们想要让首页默认是其他的html文件怎么办呢？例如我们讲我们的`index.html` 更改为 `login.html`。 简单的方法是我们可以在controller中写一个方法并且 `@RequestMapping("/")` 然后return需要作为默认页面的页面。还有一种方法是我们可以通过一个配置类来配置 SpringMVC，修改默认的主页。在 config 包下新建 `MyMvcConfig` 类。

```java
@Configuration // 申明这是一个配置类
public class MyMvcConfig {

    @Bean 
    /*
        将我们定义的 WebMvcConfigurer 作为组件加入到容器中，SpringBoot进行自动配置的过程中，如果某个Bean在容器中已经存在，那么SpringBoot就会跳过该Bean的默认自动配置，在这里我们用我们自己的 webMvcConfigurer 替换了默认的 bean
    */ 
     public WebMvcConfigurer webMvcConfigurer(){
         WebMvcConfigurer webMvcConfigurer = new WebMvcConfigurer() {
             @Override
             public void addViewControllers(ViewControllerRegistry registry) {
                registry.addViewController("/").setViewName("main");
                 registry.addViewController("/index.htm;").setViewName("main");
             }
         };
         return webMvcConfigurer;
     }
}
```

访问项目地址，你会发现默认载入的已经是 `login.html` 了。接下来还有的问题是咱们的几个html页面都是基于 Bootstrap 实现的，我们可以发现在我们的静态资源文件夹中也有和Bootstrap相关的js文件。我们之前讲过我们可以通过webjar的形式来引入静态资源，因此我们同样可以通过webjar来引入Bootstrap的相关依赖。

```
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>bootstrap</artifactId>
    <version>4.5.0</version>
</dependency>
```

接下来我们打开 login.html 修改 html 标签:

```html
<!--启用模板引擎的智能提示-->
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```

接下来我们发现 login.html 引用了我们 `asserts/css` 下的css文件，现在我们就可以通过模板引擎将这里修改为我们引入的webjar：

```html
<!-- Bootstrap core CSS -->
<link href="asserts/css/bootstrap.min.css" th:href="@{/webjars/bootstrap/4.5.0/css/bootstrap.css}" rel="stylesheet">
<!-- Custom styles for this template -->
<!--引用外部CSS或者用到相对路径都要用模板引擎替换，这样即使改变项目根目录地址，新的地址也会自动被模板引擎填写-->
<link href="asserts/css/signin.css" th:href="@{/asserts/css/signin.css}" rel="stylesheet">
```

对所有的html文件进行修改，重新访问项目地址，一切正常。

## 国际化

现实生活中我们有很多场景下需要提供App的国际化支持，SpringBoot自然也包含了这个特性。我们可以通过国际化配置文件来实现国际化。我们用登录页面 `login.html` 举例。首先，在 recourse 下创建 `i18n` 文件夹，然后添加三个文件 `login.properties`, `login_en_US.properties` 和 `login_zh_CN.properties`。如果需要支持更多的语言，则是添加更多的 `login_语言代码_区域代码.properties`。创建完成后，你会发现 Idea 已经自动将我们的配置文件归为了一个 Bundle：

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-15-19-58-42.png)

接下来选中 Bundle，并点击右下方的 Resource Bundle

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-15-20-02-24.png)

为登录页面的每一处文本添加对应的配置：

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-15-20-11-01.png)

修改好了之后，我们需要告诉SpringBoot我们的配置文件的位置，让SpringBoot使用我们的配置文件，因此在application.yml中添加:

```yaml
spring:
  messages:
    basename: i18n.login
```

值得注意的是 `spring.messages.basename` 自动配置的默认值是 `messages` 也就是说如果将 `messages.properties` 直接放在 resource 目录下，无须进行任何配置即可使用。对于较小规模的项目来说较为实用。做好了国际化相关的配置文件后，我们需要在模板中读取相应字段的值。以 Please Sign in 这个区域为例，修改 HTML 如下：

```html
<h1 class="h3 mb-3 font-weight-normal" th:text="#{login.prompt}">Please sign in</h1>
```

这样我们的模板引擎便会根据语言将标签内的值替换成 `login.prompt` 的对应语言的文本，如果浏览器的默认语言是中文的话，页面中的对应文本就会变成中文：

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-15-20-28-02.png)

现在我们的页面会根据浏览器的语言设置切换语言。但是我们如后让用户通过点击来选择语言呢？在SpringBoot中，获取需要什么语言是由 `LocaleResolver` 组件来实现的，我们可以查看 `WebMvcAutoConfigurator` 类来查看这个组件的实现：

```java
@Bean
@ConditionalOnMissingBean
@ConditionalOnProperty(prefix = "spring.mvc", name = "locale")
public LocaleResolver localeResolver() {
    if (this.mvcProperties.getLocaleResolver() == WebMvcProperties.LocaleResolver.FIXED) {
        return new FixedLocaleResolver(this.mvcProperties.getLocale());
    }
    AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
    localeResolver.setDefaultLocale(this.mvcProperties.getLocale());
    return localeResolver;
}
```

可以看到默认的 LocaleResolver 是通过浏览器发送的请求的请求头中的 `Accept-Language` 来决定语言。为了实现我们的目标，我们可以讲需要的语言作为请求参数，实现一个我们自己的 `LocaleResolver`，我们在 component 包下建立新的类 `MyLocaleResolver` 并实现 `LocaleResolver` 接口：

```java
public class MyLocaleResolver implements LocaleResolver {
    
    @Override
    public Locale resolveLocale(HttpServletRequest httpServletRequest) {
        String lang = httpServletRequest.getParameter("lang"); // 获取请求参数lang
        Locale locale = Locale.getDefault(); // 获取系统默认的Locale
        if(lang != null && lang.length() !=0){ // 若包含请求参数
            String[] splitted = lang.split("-"); // "en_US" -> ["en", "US"]
            locale = new Locale(splitted[0], splitted[1]); // 创建新的locale
        }
        return locale;
    }

    @Override
    public void setLocale(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Locale locale) {}
}
```

实现了这个组件后，我们需要将这个组件添加到容器中，我们向 `config.MyMvcConifg` 中添加配置方法：

```java
@Bean
public LocaleResolver localeResolver(){
    return new MyLocaleResolver();
}
```

最后修改我们的login.html文件，为下方的 english， 中文版添加超链接：

```html
<a class="btn btn-sm" th:href="@{/(lang=zh-CN)}">中文</a>
<a class="btn btn-sm" th:href="@{/(lang=en-US)}">English</a>
```

即可发现现在点击下方的按钮可以切换语言:

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-15-20-54-06.png)

## 登录

未完待续













