---
title: SpringBoot 日志
date: 2020-06-13 13:03:37
tags: 
    - Spring
    - SpringBoot
categories: 找工作
---

# 1. 简介

### 常见日志框架

| 日志抽象层                | 日志实现                                  |
| ------------------------- | ----------------------------------------- |
| JCL, SLF4j, jboss-logging | log4j, java.util.logging, log4j2, logback |

我们需要一个抽象层 + 一个日志实现，通常使用 SLF4j + Logback。SpringBoot本身使用的便是这种组合

<!--more-->

### SLF4j 如何使用

切记永远调用接口，不要调用具体的实现类.

```java
import org.slf4j.Logger; // 调用slf4j接口
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

### SpringBoot 中的日志

1. SpringBoot 底层使用 SLF4j + Logback
2. 为了解决各种框架日志不统一，通过适配器模式 (`jcl-over-slf4j`, `jul-over-slf4j`)将其他的日志框架调用转换为了 SLF4j
3. 引入新框架后，应该讲该框架的默认日志**排除**掉，让框架使用SpringBoot的转换

# 2. 使用

我们直接在我们的单元测试中尝试使用logger。首先修改测试类

```java
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory; // 注意倒入的是SLF4j的Logger
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class DemoApplicationTests {
    // 
    Logger logger = LoggerFactory.getLogger(getClass());
    @Test
    void contextLoads() {
        logger.trace("This is a trace message");
        logger.debug("This is a debug message");
        logger.info("This is a info message");
        logger.warn("This is a warning message");
        logger.error("This is an error message");
    }
}
```

在这里我们会发现默认日志有五个级别，从级别低到级别高分别为 trace, debug, info, warn, 和 error。SpringBoot 默认的日志记录级别是 info， 因此如果我们执行单元测试，输出结果为：

```
2020-06-13 13:42:32.383  INFO 15559 --- [           main] com.cyf.demo.DemoApplicationTests        : This is a info message
2020-06-13 13:42:32.383  WARN 15559 --- [           main] com.cyf.demo.DemoApplicationTests        : This is a warning message
2020-06-13 13:42:32.383 ERROR 15559 --- [           main] com.cyf.demo.DemoApplicationTests        : This is an error message
```

我们可以在配置文件中配置log的默认输出级别：

```yaml
logging:
  level:
    com.cyf: trace
```
这里我们把所有在 `com.cyf` 包下的日志输出级别都改为了trace，接下来我们再次运行测试：

```log
2020-06-13 13:45:59.044 TRACE 16091 --- [           main] com.cyf.demo.DemoApplicationTests        : This is a trace message
2020-06-13 13:45:59.044 DEBUG 16091 --- [           main] com.cyf.demo.DemoApplicationTests        : This is a debug message
2020-06-13 13:45:59.044  INFO 16091 --- [           main] com.cyf.demo.DemoApplicationTests        : This is a info message
2020-06-13 13:45:59.044  WARN 16091 --- [           main] com.cyf.demo.DemoApplicationTests        : This is a warning message
2020-06-13 13:45:59.044 ERROR 16091 --- [           main] com.cyf.demo.DemoApplicationTests        : This is an error message
```

此时所有的log都输出到了控制台，如果想要将日志输出的文件，我们也可以在配置文件中配置：

```yaml
logging:
  level:
    com.cyf: trace
  file:
    path: /home/virgil/spring/log
```

此时日志不但会在控制台输出，同样会保存到 `/home/virgil/spring/log` 目录下，默认的文件名为spring.log。更加具体的配置可以参考官方文档，不再赘述。


