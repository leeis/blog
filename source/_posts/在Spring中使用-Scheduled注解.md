---
title: 在Spring中使用@Scheduled注解
tags:
  - scheduled
  - annotation
categories:
  - Spring
comments: true
date: 2017-05-24 21:25:08
---


### 配置Scheduling可用

> 如果完全是使用Spring boot这种全注解的方式，要将任务调度类加入Bean中，可以使用`@Component`关键字，否则调度不会生效

要让Spring支持任务调度的注解，需要先配置`@Configuration`和`@EnableScheduling`

```java
@Configuration
@EnableScheduling
public class SpringConfig{
	...
}
```

### Fixed Delay

延迟模式：每个任务都会延迟指定时间执行，下一个任务一定要等当前任务执行完毕后才会继续延迟执行。比如配置`fixedDelay=3000`， 程序启动后**马上**执行第一个任务，第一个任务执行完毕后延迟3秒执行第二个任务，如此反复，同一时间只会有一个任务在执行，每个任务之间延迟指定时间。

```java
@Scheduled(fixedDelay = 1000)
public void scheduleFixedDelayTask() {
    ...
}
```

### Fixed Rate

周期模式：和`Fixed Delay`刚好相反，每个任务都会在指定周期后马上执行，不管上一个任务是否已经结束。

```java
@Scheduled(fixedRate = 1000)
public void scheduleFixedRateTask() {
    ...
}
```

### 带有初始延迟的定时任务(Initial Delay)

初始延迟是针对首次任务，设置了`initialDelay`之后，第一次执行的任务将会延迟指定的时间，配合`fixedDelay`
和`fixedRate`一起使用。

```java
@Scheduled(fixedDelay = 1000, initialDelay = 1000)
public void scheduleFixedRateWithInitialDelayTask() {
    ...
}
```

### 使用周期调度表达式(Cron Expressions)

```java
@Scheduled(cron = "0 15 10 15 * ?")
public void scheduleTaskUsingCronExpression() {
	...  
}
```

### 带参数的周期调度

如果不想将任务调度时间固定在代码中，可以将周期配置在properties文件中，然后在代码中通过下列方式获取。

`@Scheduled(fixedDelayString = "${fixedDelay.in.milliseconds}")`

`@Scheduled(fixedRateString = "${fixedRate.in.milliseconds}")`

`@Scheduled(cron = "${cron.expression}")`

### 在XML文件中配置

也可以在XML文件中配置

```xml
<!-- Configure the scheduler -->
<task:scheduler id="myScheduler" pool-size="10" />
 
<!-- Configure parameters -->
<task:scheduled-tasks scheduler="myScheduler">
    <task:scheduled ref="beanA" method="methodA"
      fixed-delay="5000" initial-delay="1000" />
    <task:scheduled ref="beanB" method="methodB"
      fixed-rate="5000" />
    <task:scheduled ref="beanC" method="methodC"
      cron="*/5 * * * * MON-FRI" />
</task:scheduled-tasks>
```

--- 

> 文章来自IOMan的[个人博客](http://ioman.cc)

---