---
title: SpringBoot Web 开发
date: 2020-06-16 13:03:37
tags: 
    - Spring
    - SpringBoot
categories: 找工作
toc: true
---

在写这篇笔记之前我也有很多思考，我参考的教程中有很多SpringMVC以及JSP，模板引擎的内容。貌似现在的开发大环境下，后端需要兼顾手机App和网页版共同的请求，前后端分离是大的趋势。但是我仍然准备学习一下传统的服务器端渲染的相关技术。一方面是感受一下技术的演进，另一方面是前后端分离不利于SEO（Serach Engine Optimization），很多现有的产品仍然使用服务器渲染的原因就是搜索引擎。虽然现在可能会有一些更加成熟的方案（nodejs作为中间层），但是咱们还是一步一步来。

<!--more-->

# Spring Boot 静态资源映射

在以往的SpringMVC中，我们会将静态资源文件储存在WEBAPP中，但是在SpringBoot中则有所不同。开始前使用 Spring Initializer 新建一个 restful-api 的项目。勾选 web, Spring Data JPA, MySQL 模块.

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

将我们的 html, css, js 文件导入项目。完成后目录的结构如下：

```
src
├── main
│   ├── java
│   │   └── com
│   │       └── cyf
│   │           └── restfulapi
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
                registry.addViewController("/").setViewName("login");
                 registry.addViewController("/index.htm;").setViewName("login");
             }
         };
         return webMvcConfigurer;
     }
}
```

访问项目地址，你会发现默认载入的已经是 `login.html` 了。接下来还有的问题是咱们的几个html页面都是基于 Bootstrap 实现的，我们可以发现在我们的静态资源文件夹中也有和Bootstrap相关的js文件。我们之前讲过我们可以通过webjar的形式来引入静态资源，因此我们同样可以通过webjar来引入Bootstrap的相关依赖。

```xml
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

我们首先不考虑安全的实现一个简单的登录功能，用户在页面中输入用户名和密码，如果在数据库中存在用户名和密码，则用户登录成功，进入详情页面。否则提示用户登录失败。首先我们修改我们的登录页，添加表单的提交地址，并且为username 和 password 添加 name 属性：

```html
<form class="form-signin" action="dashboard.html" th:action="@{/api/session}" method="post">
    ...
    <input type="text" class="form-control" placeholder="Username" name="username" th:placeholder="#{login.username}" required="" autofocus="">
    ...
    <input type="password" class="form-control" placeholder="Password" name="password" th:placeholder="#{login.password}" required="">
    ...
</form>
```

接下来在 entity 包下创建 User 类，加上JPA注解以及Getter，Setter：

```java
@Entity
public class User {

    @Id // 主键
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    @Column(unique = true, nullable = false) // 非空，唯一
    private String username;

    @Column(nullable = false)
    private String password;
    private String email;
}
```

在 Repository 包下创建 UserRepository 接口：

```java
public interface UserRepository extends JpaRepository<User, Integer> {
    /*

    */
    public User findByUsername(String username);
}
```

我们在接口里增加了一个方法 `findByUsername()`，这里的方法名指定了查询条件，对于这个方法，Spring Data JPA 会自动为我们生成相应的查询语句而无需我们做任何实现，具体的方法命名规则参考 [官方文档](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)。就此我们的数据访问就已经完成，在application.yml中添加数据库的相应配置以及启用generate-ddl，运行项目，让Spring Data自动为我们在数据库中创建User表。在User表中添加一个用户用于测试：

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-17-22-27-02.png)

接下来编写Controller，在controller包下创建 `LoginController`:

```java
@Controller
@RequestMapping("/api")
public class LoginController {

    @Autowired
    UserRepository userRepository;

    @PostMapping("/session")
    public String login(@RequestParam("username") String username,
                        @RequestParam("password") String password,
                        Map<String, String> map){
        // 根据用户名获取用户
        User loginUser = userRepository.findByUsername(username);
        // 用户存在 对比密码
        if(loginUser!= null && loginUser.getPassword().equals(password)){
            return "dashboard";
        }else{
            map.put("msg", "用户名密码错误");
            return "login";
        }
    }
}
```

然后在 `login.html` 中的Please Sign in 下添加如下一行，添加密码错误时的错误提示：

```html
...
<p style="color: darkred" th:text="${msg}" th:if="${not #strings.isEmpty(msg)}"></p>
...
```

现在尝试登录，访问 `localhost:8080`，并输入错误的用户名和密码：

![](img/(2020-06-14)SpringBoot-Web开发.md/2020-06-17-22-34-00.png)

再尝试正确的密码，发现成功跳转到了dashboard页面。

## 拦截器

现在如果我们在dashboard界面刷新，你会发现浏览器提示我们是否需要重新提交表单。这是因为刷新时浏览器会重新发送我们登录时的Post请求。为了防止重复提交，我们修改为重定向的方式来让用户到达dashboard页面。观察我们之前的请求，用户登录成功后，浏览器的url栏显示的时我们post地址的url，而修改为重定向后，浏览器则会跳转到我们重定向的url。

首先修改 `HelloController`：

```java
if(loginUser!= null && loginUser.getPassword().equals(password)){
    session.setAttribute("loginUser", loginUser.getId());
    // return "dashboard"
    return "redirect:/main";
}
```

在用户登录成功后，我们将用户重定向到 `/main`，也就是说浏览器会向 `/main` 发送一次请求，因此我们将 dashboard视图绑定到 `/main`, 修改 `MyMvcConfig` 配置类中我们之前实现的 `WebMvcConfigurator`：

```java
public void addViewControllers(ViewControllerRegistry registry) {
    registry.addViewController("/").setViewName("login");
    registry.addViewController("/index.html").setViewName("login");
    // 重定向 /main 到视图 dashboard
    registry.addViewController("/main").setViewName("dashboard");
}
```

此时再次登录，你会发现浏览器会自动跳转到 `localhost:8080/main`，并且刷新浏览器也不会再提示需要重新提交表单。但是此时你会发现一个问题，就是如果此时我们在其他浏览器中直接访问 `localhost:8080/main`，便跳过了登陆直接进入了dashboard，这是我们不希望看到的，因此我们需要拦截那些未登录用户的请求，这里就用到了拦截器。首先我们要做的是识别一个用户的登录状态，修改 `LoginContoller` 中的 login 方法：

```java
@PostMapping("/session")
public String login(@RequestParam("username") String username,
                    @RequestParam("password") String password,
                    Map<String, String> map,
                    // 传入session
                    HttpSession session){
    User loginUser = userRepository.findByUsername(username);
    if(loginUser!= null && loginUser.getPassword().equals(password)){
        // 将登录用户的id保存在session中
        session.setAttribute("loginUser", loginUser.getId());
        return "redirect:/main";
    }else{
        map.put("msg", "用户名密码错误");
        return "login";
    }
}
```

此时我们就可以判断session中是否有 `loginUser` 这个attribute来判断用户是否登录，接下来我们就可以开始编写我们的拦截器，在 component 包下新建 `LoginHandlerInterceptor` 类:

```java
public class LoginHandlerInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 取出 loginUser
        Object loginUserId = request.getSession().getAttribute("loginUser");
        if(loginUserId != null){
            // 若存在，则用户已登录，放行请求
            return true;
        }else{
            // 若不存在，重定向用户到登录页面
            request.getRequestDispatcher("/index.html").forward(request, response);
        }
        return false;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```

该类实现了 `HandlerInterceptor` 接口，这个接口有三个方法，当容器收到一个请求后，如果满足条件，则会在controller方法执行前先执行 `preHandle` 方法，该方法的返回值是一个布尔值，代表的是是否拦截该请求。接下来修改配置类，将我们自定义的拦截器添加进容器。修改 `MyMvcConfig` 类，之前我们在该类中添加了一个返回 `WebMvcConfigurator` 的方法，我们在该方法中编写了一个 `WebMvcConfigurator` 的匿名实现类用于重定向的配置，现在我们为这个类添加一个新的方法：

```java
public WebMvcConfigurer webMvcConfigurer(){
    WebMvcConfigurer webMvcConfigurer = new WebMvcConfigurer() {
        @Override
        public void addViewControllers(ViewControllerRegistry registry) {
            registry.addViewController("/").setViewName("login");
            registry.addViewController("/index.html").setViewName("login");
            registry.addViewController("/main").setViewName("dashboard");
        }

        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(new LoginHandlerInterceptor())
                    // 拦截所有请求
                    .addPathPatterns("/**") 
                    // 排除登录页面和登录请求的拦截
                    .excludePathPatterns("/index.html","/","/api/session")
                    // 排除静态资源的拦截
                    .excludePathPatterns("/asserts/**", "/webjars/**");
        }
    };
    return webMvcConfigurer;
}
```

这样未登录的用户就只能登录请求页面和发送登录请求，对于服务器其他url的访问都会被拦截器拦截，并且强制跳转到登录页面。现在，我们再尝试在另一个浏览器访问 `localhost:8080/main`，会发现直接跳转到了登录界面。

## REST API 和页面

### 1. API

为系统设计一下 RESTful API：

| 功能         | URI                | 请求方式 |
| ------------ | ------------------ | -------- |
| 查询所有员工 | /employees     | GET      |
| 查询一个员工 | /empoyee/{id}  | GET      |
| 添加员工     | /employee      | POST     |
| 修改员工     | /employee      | PUT      |
| 删除员工     | /employee/{id} | DELETE   |

来到 dashboard 页面，我们希望在点击 customer 这个链接时跳转到员工列表，并可以对员工进行增删查改

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-20-15-02-50.png)

修改 dashboard.html, 修改customer按钮的链接以及文本，注意要使用模板引擎：

```html
<a class="nav-link" href="#" th:href="@{/employee}">
    ...
    Employees
</a>
```

### Themyleaf 公共元素提取

同时我们注意到员工页面和dashboard页面其实侧边栏和顶栏使用的都是相同的元素，通过观察我们发现在dashboard页面，侧边栏的所有DOM在:

```html
<nav class="col-md-2 d-none d-md-block bg-light sidebar">
```

这个标签下，因此我们我们可以通过thymeleaf提取这个公共组件，再插入到dashboard页面中去，这样我们就不需要在list页面把侧边栏和顶栏的html都复制一遍了，我们可以通过添加 `id` 或是通过添加 `th:fragment` 这两个属性来标注一个公共元素：

```html
<nav class="col-md-2 d-none d-md-block bg-light sidebar" id="sidebar">
<!--or-->
<nav class="col-md-2 d-none d-md-block bg-light sidebar" th:fragment="sidebar">
```

这样我们就可以去 list.html，讲list.html文件中重复的侧边栏DOM替换为：

```html
<div th:replace="dashboard :: #sidebar"> <!--使用css选择器-->
<div th:replace="dashboard :: sidebar"> <!--使用fragment标注-->
```

完成后，为了测试效果，我们编写一个简单的 `EmployeeController`：

```java
@Controller
public class EmployeeController {
    @GetMapping("/employee")
    public String list(Model model){
        return "list";
    }
}
```

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-20-20-44-50.png)

试着访问list页面，发现侧边栏依然可以正常显示。因此我们可以吧这些通用的HTML组件提取出来。在 templates 下新建 commons 文件夹，创建 bar.html，把我们之前标注的sidebar部分的HTML剪切过来。同时我们对于顶栏也可以做相同的操作：

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-20-20-56-06.png)

然后在dashboard和list页面使用 `th:replace` 来注入top bar和side bar：

```html
<div th:replace="commons/bar :: sidebar"/>
```

但是仍然有一个问题，就是即使我们现在处于Employee页面，侧边栏高亮的仍然是 Dashboard。这里我们需要用到themyleaf的参数化片段。观察高亮的按钮，我们发现其实是标签的class多了一个 `active`:

```html
<a class="nav-link active" href="#" th:href="@{/main}">
    ...
    Dashboard 
</a>
```

我们可以在此处添加一个判断，通过判断 `activeUri` 来决定是否要在class中加入 `active`：

```html
<a class="nav-link active" href="#" th:class="${activeUri == 'main' ? 'nav-link active' : 'nav-link'}" th:href="@{/main}">
    ...
    Dashboard 
</a>
...
<a class="nav-link active" href="#" th:class="${activeUri == 'employee' ? 'nav-link active' : 'nav-link'}" th:href="@{/employee}">
    ...
    Employee
</a>
```

那么这个 `activeUri` 的变量从哪里来呢？这是我们就要修改dashboard和list这两个页面中引用这段HTML的地方，传入 `activeUri`：

```html
<!--dashboard-->
<div th:replace="commons/bar :: sidebar(activeUri='main')"/>
<!--dashboard-->
<div th:replace="commons/bar :: sidebar(activeUri='employee')"/>
```

此时侧边栏就可以根据页面切换高亮了。

### 填充员工数据

接下来我们为员工创建实体类和相对应的Repository，不记得如何编写的可以参考之前数据访问的笔记。在数据库中插入一些假数据。然后编写 EmployeeController：

```java
@Controller
public class EmployeeController {

    @Autowired
    EmployeeRepository employeeRepository;

    @GetMapping("/employee")
    public String list(Model model){
        List<Employee> employees= employeeRepository.findAll();
        model.addAttribute("employees", employees);
        return "list";
    }
}
```

接下来修改 list 页面的表格部分，使用模板引擎进行替换：

```html
<div class="table-responsive">
    <table class="table table-striped table-sm">
        <thead>
            <tr>
                <th>#</th>
                <th>LastName</th>
                <th>Email</th>
                <th>Gender</th>
                <th>Department Id</th>
                <th>Birth</th>
            </tr>
        </thead>
        <tbody>
            <tr th:each="employee:${employees}">
                <td th:text="${employee.id}"/>
                <td th:text="${employee.lastName}"/>
                <td th:text="${employee.email}"/>
                <td th:text="${employee.gender} == 0 ? 'Male' : 'Female'"/>
                <td th:text="${employee.departmentId}"/>
                <td th:text="${#dates.format(employee.birth, 'dd/MM/yyyy')}"/>
            </tr>
        </tbody>
    </table>
</div>
```

现在访问员工管理页面，发现所有员工的数据已经正常展示出来了：

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-22-14-34-21.png)

### 员工添加

现在已经可以列出所有员工，现在我们添加增加员工的功能。首先在list界面添加相关的按钮以及他们对应的操作地址，添加员工页面的地址我们定为 `GET /addEmployee` :

```html
<main role="main" class="col-md-9 ml-sm-auto col-lg-10 pt-3 px-4">
    <!-- 添加按钮 -->
    <a class="btn btn-sm btn-primary" href="/addEmployee" th:href="@{/addEmployee}">Add</a>
    ...
</main>
```

界面大概会变成这样：

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-22-14-43-00.png)

然后在 EmployeeController 中添加方法：

```java
// 返回添加员工页面
@GetMapping("/addEmployee")
public String toAddPage(Model model){
    // 添加员工页面需要选择部门，因此把所有部门查找出来放入model
    List<Department> departments = departmentRepository.findAll();
    model.addAttribute("departments", departments);
    return "add";
}

@PostMapping("/employee")
// Spring 会自动把收到的参数parse到Employee对象中
public String addEmployee(Employee employee){
    employeeRepository.save(employee);
    // 重定向到员工列表
    return "redirect:/employee";
}
```

接下来我们需要增加一个添加员工的模板，把 list.html 复制一份，删除main标签内的所有内容，编写一个和 Employee 类对应的表单，注意所有 `<input>` 标签一定要有 `name` 属性，该属性的值为 Employee 对象内对应的属性名：

```html
<form th:action="@{/employee}" method="post">
    <div class="form-group">
        <label for="lastname">Last Name</label>
        <!--Employee对象中属性名为lastName，因此name就是lastName-->
        <input name="lastName" type="text" class="form-control" id="lastname" placeholder="Bob">
    </div>
    <div class="form-group">
        <label for="email">Email</label>
        <input name="email" type="email" class="form-control" id="email" placeholder="name@example.com">
    </div>
    <div class="form-group">
        <label for="gender">Gender</label>
        <div id="gender" >
            <div class="form-check form-check-inline">
                <input class="form-check-input" type="radio" name="gender" id="inlineRadio1" value="0">
                <label class="form-check-label" for="inlineRadio1">Male</label>
            </div>
            <div class="form-check form-check-inline">
                <input class="form-check-input" type="radio" name="gender" id="inlineRadio2" value="1">
                <label class="form-check-label" for="inlineRadio2">Female</label>
            </div>
        </div>
    </div>
    <div class="form-group">
        <label for="department">Department</label>
        <select multiple class="form-control" id="department" name="departmentId">
            <!--取出model中的Department，循环创建对应option-->
            <!--提交到数据库中的为DepartmentId，因此value="${department.id}"-->
            <option th:each="department:${departments}" th:text="${department.departmentName}" th:value="${department.id}"></option>
        </select>
    </div>
    <div class="form-group">
        <label for="birth">Birth</label>
        <input type="text" name="birth" class="form-control" id="birth" placeholder="2000-01-01">
    </div>
    <button type="submit" class="btn btn-primary">Submit</button>
</form>
```

完成后，点击添加按钮，我们可以来到添加员工的页面：

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-22-19-02-21.png)

填入相关信息（注意日期格式为 yyyy/MM/dd），点击 Submit 按钮，我们自动跳转到了员工列表页面，并且新的员工也出现在了列表中：

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-22-19-03-25.png)

## 错误处理

为了方便测试，我们先关闭拦截器，注释掉相应代码。如果此时访问一个项目中不存在的地址，我们会得到一个默认的错误页面：

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-19-19-44-54.png)

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























