---
title: SpringBoot 配置文件
date: 2020-06-12 17:49:12
tags: Spring, SpringBoot
categories: 找工作
---

# Spring Boot 配置

## 1. yaml 语法

**基本语法**：

- 使用缩进表示层级关系
- 缩进使用空格
- 大小写敏感

```yaml
server:
    port: 8080
    address: www.example.com
```

**支持的数据结构**

1. 字面量
   
    字符串无需添加双引号
    ```yaml
    address: www.example.com
    ```

2. 对象和Map
    
    多行写法
    ```yaml
    people: 
        name: Frank
        age: 18
        gender: male
    ```

    行内写法
        多行写法
    ```yaml
    people: {name: Frank, age: 18, gender: male}
    ```

3. 数组

    多行写法
    ```yaml
    animals:
        - cat
        - dog
        - pig
    ```

    行内写法
        多行写法
    ```yaml
    people: {cat, dog, pig}
    ```

## 2. 获取配置文件的值 （@ConfigurationProperties）

首先我们在 `com.cyf.demo.bean` 包下创建一个新的 `Hero` 类：

```java
// 省略了 getter, setter和toString
public class Hero {
    // 测试字面量的获取
    private String name;
    private int atk;
    private int def;
    private Boolean isAlive;

    // 测试Map的获取
    private Map<String, Integer> abilities;

    // 测试数组的获取
    private List<String> equipments;
    
    // 测试对象的获取
    private Weapon weapon;
}
```

同样的我们创建 `Hero` 类包含的 `Weapon` 类

```java
// 省略了 getter, setter和toString
public class Weapon {
    private int atk;
    private int def;
}
```

此时我们在 `src/main/resources` 文件夹下创建 `application.yml`，并添加以下内容, 可以发现与我们定义的 `Hero` 类的属性完全一致：

```yaml
Hero: 
  name: Ashe
  atk: 20
  def: 20
  isAlive: ture
  abilities:
    bomb: 20
    shotgun: 10
  weapon:
    atk: 20
    def: 0
```

然后我们为 `Hero` 类添加 `@ConfigurationProperties` 注解。它可以讲配置文件中的属性值和类中对应的属性值联系起来。

```java
@Component // 启用配置文件功能，类本身必须是一个Component
@ConfigurationProperties(prefix = "Hero")
public class Hero {
    ...
}
```

此时我们得到了一条提示：

![](img/2020-06-12-18-09-24.png)

这是因为我们并没有在 Maven 中添加 SpringBoot 的注解处理器，根据官方文档，我们在 `pom.xml` 中添加：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

然后重新运行一遍我们的 `DemoApplication`, Annotation Processor 就会正常工作了, 编写配置文件的时候Idea也会进行提示。

接下来我们修改 `src/test/java/com/cyf/demo/DemoApplicationTests.java`, 编写一个单元测试来测试我们配置文件的值是否注入。

```java
@SpringBootTest
class DemoApplicationTests {
    // 自动注入Hero类
    @Autowired
    Hero hero;
    @Test
    void contextLoads() {
        System.out.println(hero);
    }
}
```

运行测试，我们发现输出为：

```
Hero{name='Ashe', atk=20, def=20, isAlive=null, abilities={bomb=20, shotgun=10}, equipments=null, weapon=Weapon{atk=20, def=0}}
```

也就是说Spring成功的获取了我们配置文件中的值并注入到了对应的类中。

## 3. 获取配置文件的值 （@Value）

使用 `@Value` 注解和在之前的 Spring 中配置文件的 `<bean>` 便签中进行配置基本相同，原来我们会这样配置一个bean：

```xml
<bean class="Hero">
    <property name="name" value="()">
<bean/>
```

在value处Spring支持三种类型：
1. 字面量
2. `${key}` 从配置文件中获取key对应的value
3. `#{SpEL}` 运算

因此 `@Value` 注解同样支持这三种类型。作为测试，我们修改Hero类如下：

```java
// 省略了 getter, setter和toString
@Component
//@ConfigurationProperties(prefix = "hero") 不再使用 ConfigurationProperties
public class Hero {
    @Value("${hero.name}") // 通过key获取value
    private String name;
    @Value("#{10*2}") // 计算
    private int atk;
    @Value("20") // 字面量
    private int def;
    private Boolean isAlive;
    private Map<String, Integer> abilities;
    private List<String> equipments;
    private Weapon weapon;
}
```

重新运行单元测试，输出为：

```
Hero{name='Ashe', atk=20, def=20, isAlive=null, abilities=null, equipments=null, weapon=null}
```
 
可以发现我们注解了的三个值都成功的注入到了对象中。

## 4. @Value 和 @ConfigurationProperties 的区别

|           | `@Value`             | `@ConfigurationProperties` |
| --------- | -------------------- | -------------------------- |
| 功能      | 批量注入配置文件属性 | 单个属性注入               |
| 松散绑定* | 支持                 | 不支持                     |
| SpEL      | 不支持               | 支持                       |
| 复杂类型  | 支持                 | 不支持                     |

*松散绑定指的是 `lastName` = `last-name`

## 5. @PropertySource 和 @ImportResource

如果所有配置都写在 `application.properties` 里，会显得配置文件过于复杂或者庞大。此时我们可以使用 `@PropertySource` 来指定要读取的配置文件。刚才我们把所有的 `Person` 类的属性写在了 `application.yml` 里。现在我们可以把内容改成 `properties` 的格式，然后放入 `person.properties` 文件中：

```properties
hero.name=Ashe
hero.atk=20
hero.def=20
hero.isAlive=ture
hero.abilities.bomb=20
hero.abilities.shotgun=10
hero.weapon.atk=20
hero.weapon.def=0
```

然后再 `Hero` 类前方添加如下注解：

```java
@PropertySource("classpath:person.properties") // 使用 person.properties 作为本类的配置文件
@Component
@ConfigurationProperties(prefix = "Hero")
public class Hero {
    ...
}
```

配置一样可以成功注入类中。

而 `@ImportResource` 则是用来加载自定义配置文件用的，如果我们在 `resource` 目录下添加一个 `beans.xml` 文件，SpringBoot 不会自动导入这个配置文件，我们需要在 `DemoApplications` 中加入注解：

```java
@ImportResource(locations={"classpath:bean.xml"})
```

SpringBoot 才会识别这个配置文件。但是SpringBoot并不推荐我们这样添加新的组件，推荐使用**全注解**的方式来添加组件。

首先在我们的 `com.cyf.demo` 包下创建一个新的 `service` 包。然后编写一个 `HelloService` 类。

```java
package com.cyf.demo.service;

public class HelloService {
    public void sayHello(){
        System.out.println("This is a hello from hello service");
    }
}

```

当我们尝试在单元测试中注入 `HelloService` 时， 会发选 Idea 已经提示我们无法找到这个bean：

![](/img/2020-06-12-21-32-51.png)

通常我们会编写一个配置文件来导入这个包，但是现在我们通过配置类的方式。在我们的 `com.cyf.demo` 包下创建一个新的 `config` 包，新建 `HelloConfig` 类：

```java
@Configuration // 表明这是一个配置类
public class HelloConfig {
    
    @Bean // 等同于原本的<bean> 标签，方法名就是原本的bean id
    public HelloService helloService(){
        return new HelloService();
    }
}
```

此时我们再运行单元测试：

```
This is a hello from hello service
```

这个组件就通过我们的配置类加载进来了。

## 6. 配置文件占位符

共有两种，一种是随机数占位符，还有一种时属性配置占位符。我们分别举例。

之前我们的 `application.yml` 中指定了 `Hero` 类型的一些属性，现在我们可以向其中添加一些占位符：

```yaml
hero:
  name: Ashe${random.uuid} # 生成随机UUID
  atk: ${random.int} # 生成随机整数
  def: 20
  isAlive: ture
  abilities:
    bomb: 20
    shotgun: 10
  weapon:
    atk: ${hero.atk} # 使用存在的属性配置
    def: ${hero.at:10} # 若属性不存在则可以使用冒号指定默认值
```

此时我们再次运行单元测试：

```
Hero{name='Ashe8a1189c9-2740-4b6e-b85b-14de42bdfb42', atk=-156092609, def=20, isAlive=null, abilities={bomb=20, shotgun=10}, equipments=null, weapon=Weapon{atk=-499134195, def=0}}
```

可以观察到生成的UUID，随机整数的ATK，以及和ATK相同的Waepon ATK。由于 `hero.at` 这个属性不存在，`weapon.def` 被设置成了默认值10。

## Profile

在实际开发过程中经常牵涉到不同环境的切换（生产/开发/测试）。








