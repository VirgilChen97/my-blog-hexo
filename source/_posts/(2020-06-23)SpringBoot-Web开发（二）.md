---
title: SpringBoot-Web开发（二）
date: 2020-06-18 23:31:51
tags: 
    - Spring
    - SpringBoot
categories: 找工作
toc: true
---

第二部分我们来实现员工的CRUD：

# REST API

为系统设计一下 RESTful API：

| 功能         | URI                | 请求方式 |
| ------------ | ------------------ | -------- |
| 查询所有员工 | /employees     | GET      |
| 查询一个员工 | /empoyee/{id}  | GET      |
| 添加员工     | /employee      | POST     |
| 修改员工     | /employee      | PUT      |
| 删除员工     | /employee/{id} | DELETE   |

<!-- more -->

来到 dashboard 页面，我们希望在点击 customer 这个链接时跳转到员工列表，并可以对员工进行增删查改

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-20-15-02-50.png)

修改 dashboard.html, 修改customer按钮的链接以及文本，注意要使用模板引擎：

```html
<a class="nav-link" href="#" th:href="@{/employee}">
    ...
    Employees
</a>
```

## Themyleaf 公共元素提取

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

## 填充员工数据

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

## 员工添加

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

## 员工修改

我们可以在重用添加员工页面，不同的是如果是员工修改，那么表单中应该显示该员工原本的数据。首先为员工修改编写 controller:

```java
// 转到员工修改页面
@GetMapping("/employee/{id}")
public String toEditPage(@PathVariable Integer id, Model model){
    // 查找出员工和所有的部门
    Employee employee = employeeRepository.findById(id).get();
    model.addAttribute("employee", employee);
    List<Department> departments = departmentRepository.findAll();
    model.addAttribute("departments", departments);
    // 重用 Add 页面
    return "add";
}

@PutMapping("/employee/{id}")
public String editEmployee(@PathVariable Integer id, Employee employee, Model model){
    // 因为是修改，我们需要把Path中的id添加到员工对象中
    employee.setId(id);
    employeeRepository.save(employee);
    return "redirect:/employee";
}
```
然后则是要对 add 页面进行修改，让他兼容员工修改的功能：

```html
<!-- 判断页面中是否有employee这个key，从而实现不同的请求路径 -->
<form th:action="${employee != null} ? @{/employee/} + ${employee.id} : @{/employee}" method="post">
    <!-- HTML 表单不支持PUT请求，Spring通过添加下方的标签来实现解析PUT请求 -->
    <input type="hidden" name="_method" value="put" th:if="${employee!=null}"/>
    <div class="form-group">
        <label for="lastname">Last Name</label>
        <!-- 设置value为employee的值，所有操作都要事先判断是否为空 -->
        <input name="lastName" type="text" class="form-control" id="lastname" placeholder="Bob"
                th:value="${employee!=null} ? ${employee.lastName}">
    </div>
    <div class="form-group">
        <label for="email">Email</label>
        <input name="email" type="email" class="form-control" id="email" placeholder="name@example.com"
                th:value="${employee!=null} ? ${employee.email}">
    </div>
    <div class="form-group">
        <label for="gender">Gender</label>
        <div id="gender" >
            <div class="form-check form-check-inline">
                <input class="form-check-input" type="radio" name="gender" id="inlineRadio1" value="0"
                        th:checked="${employee!=null} ? ${employee.gender == 0}">
                <label class="form-check-label" for="inlineRadio1">Male</label>
            </div>
            <div class="form-check form-check-inline">
                <input class="form-check-input" type="radio" name="gender" id="inlineRadio2" value="1"
                        th:checked="${employee!=null} ? ${employee.gender == 1}">
                <label class="form-check-label" for="inlineRadio2">Female</label>
            </div>
        </div>
    </div>
    <div class="form-group">
        <label for="department">Department</label>
        <select multiple class="form-control" id="department" name="departmentId">
            <option th:each="department:${departments}"
                    th:text="${department.departmentName}"
                    th:value="${department.id}"
                    th:selected="${employee!=null} ? ${employee.departmentId == department.id}"
            ></option>
        </select>
    </div>
    <div class="form-group">
        <label for="birth">Birth</label>
        <input type="text" name="birth" class="form-control" id="birth" placeholder="2000/01/01"
                th:value="${employee!=null} ? ${#dates.format(employee.birth, 'yyyy/MM/dd')}">
    </div>
    <button type="submit" class="btn btn-primary">Submit</button>
</form>
```

接下来我们要修改Spring Boot的配置文件，启用从 hidden 的input标签解析请求方式的功能：

```yaml
spring:
  mvc:
    hiddenmethod:
      filter:
        enabled: true
```

大功告成，现在点击员工旁的edit按钮，可以看到来到了add页面，而且输入框中预先填写了员工的当前数据：

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-23-20-33-02.png)

修改这个员工的邮箱为 `frank@test.com.cn` 点击提交，可以看到回到了员工列表页面，并且员工的信息已经显示为修改后的值：

![](/img/(2020-06-14)SpringBoot-Web开发.md/2020-06-23-20-34-28.png)

## 员工删除

第一步仍然是编写 Controller：

```java
@DeleteMapping("/employee/{id}")
public String deleteEmployee(@PathVariable Integer id){
    employeeRepository.deleteById(id);
    return "redirect:/employee";
}
```

然后修改页面的删除按钮，为了避免一个按钮一个表单的臃肿，我们将表单和按钮分开，给按钮添加监听并用JavaScript来提交表单，首先修改删除按钮：

```html
<!-- 使用th:attr添加了一个员工id的参数，用于区分是哪个按钮被点击了，同时在class中添加del-btn -->
<button type="submit" class="btn btn-sm btn-danger del-btn" th:attr="employee-id=${employee.id}">Delete</button>
```

然后在 `<main>` 标签外添加一个 form：

```html
<form method="post" id="del-form">
	<input type="hidden" name="_method" value="delete">
</form>
```

然后在 `<body>` 标签下添加脚本：

```html
<script>
    // 旧版本写法
    $(".del-btn").click(function(){
        $("#del-form").attr("action", "/employee/" + $(this).attr("employee-id")).submit();
    })
    // ES6 写法
    $(".del-btn").click((event)=>{
        $("#del-form").attr("action", "/employee/" + $(event.target).attr("employee-id")).submit();
    })
</script>
```

完成后重新启动项目，点击删除按钮，可以发现员工成功的被删除了。
