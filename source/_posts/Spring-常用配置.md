---
title: Spring 常用配置
tags: 
	- Spring
comments: true
date: 2017-11-15 23:37:16
categories:
	- Spring
---


## Bean的Scope

Scope描述的是Spring容器如何新建Bean实例的。Spring的Scope通过`@Scope`注解来实现。包含以下几种方式：

- @Singleton : 一个Spring容器只有一个Bean的实例，此为默认配置
- @Prototype: 每次调用新建一个Bean的实例
- @Request: Web项目中，每个http request新增一个Bean实例
- @Session: Web项目中, 每个http session新建一个Bean实例
- @GlobalSession: 这个只在portal应用中有, 给每一个global http session新建一个Bean实例

## Spring EL表达式

spring的EL-Spring表达式主要用于`@Value`的参数中使用，包含以下几种情况：

**注入普通字符**

```
@Value(“I Love You!”)
private String normal;
```
	
**注入操作系统属性**

```
@Value(“#{systemProperties[‘os.name’]}”)
private String osName;
```

**注入表达式运算结果**

```
@Value(“#{ T(java.lang.Math).random() * 100.0 }”)
private double randomNumber;
```

**注入其他Bean的属性**

```
@Value(“#{demoService.another}”)
private String fromAnother;
```

**注入文件内容**

```
@Value(“classpath:com/lewis/el/test.txt”)
private Resource testFile;
```

**注入网址内容**

```
@Value(“http://www.baidu.com”)
private Resource testUrl;
```

**注入属性文件**

```
@Value(“${book.name}”)
private String bookName;
```