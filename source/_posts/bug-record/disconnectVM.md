---
title: 解决Disconnected from the target VM
cover: images/cover/strawberry.jpg
categories:
  - [Bug记录]
---

# 问题背景及解决方法

## 背景说明

同样是在搭建项目时出现的问题，这个问题缘由很多，解决方法也各有不同，在此总结一下，供自己和大家查阅。

## 问题缘由及解决方法

### 1、端口占用

这个问题出现之后，第一想法便是以为 127.0.0.1:XXXXX 中的端口号被占用。
**解决方法：**
即可以查找该端口：

```cmd
netstat -ano|findstr 端口号
```

如果该端口没有被占用，则什么都不会显示；
如果该端口被占用，会有至少一个连接信息，一般找到是 LISTENING 的网络连接以及其后一列的进程号 PID。

```cmd
tasklist|findstr 找到的PID（从进程列表中查找包含指定字符串的进程）
```

```cmd
taskkill /pid 找到的PID -f（根据PID杀死进程）
```

## 2、IDE 缓存累积

IDE 缓存太多，有冲突和冗余。
**解决方法：**
打开 idea 后，尝试 reload maven，或者 clean and install

```shell
File -> Invalidate Caches -> Invalidate and Restart
```

## 3、idea 里 maven 版本冲突

尽量选择 3.6.3 版本的 maven，更成熟稳定

## 4、类名重复

也是不少人的问题，有的时候项目越写越丰富，里面的类也越来越多，一定要仔细检查，解决方法就是**改类名**，不要有重复的就是了。

## 5、依赖缺失（我掉的坑）

因为 spring cloud 项目的 parent 包里缺少相关依赖。
**解决方法：**

```java
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

版本管理也加上：

```java
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <version>3.0.0</version>
</dependency>
```
