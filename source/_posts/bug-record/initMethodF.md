---
title: 解决init method failed问题

cover: images/cover/cake1.jpg

categories:
  - [Bug记录]
---

# 错误背景

笔者在建一个新的 Spring cloud 项目时，出现的报错提示如下所示：

```java
Invocation of init method failed； nested exception is java.lang.IllegalArgumentException: Property
```

# 错误原因

经过网上搜索后，主要说是 controller 和 xml 有重名方法，或者启动类有问题，如@SpringBootApplication()的括号里不能有东西。
检查后，锁定到是我在 config 类里已经写了@MapperScan()的注解：

```java
@Configuration
@EnableTransactionManagement
@MapperScan("com.demo.service.mapper")
public class MyBatisPlusConfig {
}
```

# 解决方法

**那么启动类里的@MapperScan()需要更改成@ComponentScan()。**
启动类里修改为：

```java
@SpringBootApplication
@ComponentScan("com.demo.service.mapper")
public class EduApplication {
    public static void main(String[] args) {
        SpringApplication.run(EduApplication.class, args);
    }
}
```

或者直接把 config 的@MapperScan 删了，但感觉会失去配 config 的一些意义

修改后，即不再报错，但是项目还有点其他小问题

# 总结

## @ComponentScan

组件扫描注解，用来扫描@Controller @Service @Repository 这类,主要就是定义扫描的路径从中找出标志了需要装配的类到 Spring 容器中

## @MapperScan

扫描 mapper 类的注解,就不用在每个 mapper 类上加@MapperScan 了

## 注意

这两个注解是可以同时使用的，但是需要改为@MapperScan（basePackages = {}）的形式。
或者只使用@MapperScan（）去扫描 mapper 包，让项目启动自己去扫描 swagger 配置类的包。
**同一个类扫描的是同一路径会产生错误。**
