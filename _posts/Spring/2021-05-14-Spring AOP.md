---
layout:     post
title:      "Spring - AOP实现方式"
subtitle:   " \"Spring Knowledge - 08\""
date:       2021.05.14 12:29:00
author:     "Wuy"
header-img: "img/post-bg-2020.jpg"
catalog: true
tags:
    - 学习笔记
    - 读书
    - Spring

---

> *"Keep Learning Spring"*

# AOP

## 什么是AOP

AOP(Aspect-Oriented Programming:面向切面编程)能够将那些与业务无关，**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码**，**降低模块间的耦合度**，并**有利于未来的可拓展性和可维护性**。

**Spring AOP就是基于动态代理的**，如果要代理的对象，实现了某个接口，那么Spring AOP会使用`JDK Proxy`，去创建代理对象，而对于没有实现接口的对象，就无法使用 `JDK Proxy` 去进行代理了，这时候Spring AOP会使用`Cglib` ，这时候Spring AOP会使用 `Cglib` 生成一个被代理对象的子类来作为代理，如下图所示：

![image-20210514180755274](https://i.loli.net/2021/05/14/d8GHAIVibW1zqLf.png)

当然你也可以使用 `AspectJ` ,Spring AOP 已经集成了`AspectJ` ，`AspectJ` 应该算的上是 Java 生态系统中最完整的 AOP 框架了。

使用 AOP 之后我们可以把一些通用功能抽象出来，在需要用到的地方直接使用即可，这样大大简化了代码量。我们需要增加新功能时也方便，这样也提高了系统扩展性。日志功能、事务管理等等场景都用到了 AOP 。

## 使用Spring实现AOP

首先需要导入依赖包：

```xml
<dependency>
     <groupId>org.aspectj</groupId>
     <artifactId>aspectjweaver</artifactId>
     <version>1.9.4</version>
</dependency>
```

### 方法一：使用原生的Spring API接口

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">


    <!--register bean-->
    <bean id="userService" class="com.example.learnspring.demo02.UserServiceImp"/>
    <bean id="log" class="com.example.learnspring.demo02.Log"/>
    <bean id="afterLog" class="com.example.learnspring.demo02.AfterLog"/>
    <!--方式一：使用原生的Spring API接口-->
    <!--配置aop：需要导入aop的约束-->
    <aop:config>
        <!--切入点:expression:表达式, execution(要执行的位置！)-->
        <aop:pointcut id="pointcut" expression="execution(* com.example.learnspring.demo02.UserServiceImp.*(..))"/>

        <!--执行环绕增加-->
        <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
        <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut"/>
    </aop:config>

</beans>
```

测试类：

```java
package com.example.learnspring;

import com.example.learnspring.demo02.UserService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MyTest {
    public static void main(String[] args) {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        // 动态代理代理的是接口
        UserService userService = applicationContext.getBean("userService", UserService.class);

        userService.add();
    }
}
```

输出：

```
com.example.learnspring.demo02.UserServiceImp:add executed
add a user
Having executed: addand return: null
```

### 方式二：使用自定义类

```xml
<!--方式二：自定义类-->
    <bean id="diy" class="com.example.learnspring.demo02.DiyPointCut"/>

    <aop:config>
<!--自定义切面， ref要引用的类-->
        <aop:aspect ref="diy">
<!--切入点-->
            <aop:pointcut id="point" expression="execution(* com.example.learnspring.demo02.UserServiceImp.*(..))"/>
<!--通知-->
            <aop:before method="before" pointcut-ref="point"/>
            <aop:after method="after" pointcut-ref="point"/>
        </aop:aspect>
    </aop:config>
```

自定义切面的引用类：

```java
package com.example.learnspring.demo02;

public class DiyPointCut {
    public void before() {
        System.out.println("before : ....");
    }

    public void after() {
        System.out.println("after: ....");
    }
}
```

测试方法同上，输出如下：

```
before : ....
add a user
after: ....
```

### 方法三：使用注解实现

```xml
<!--方式三-->
    <bean id="annotationPointCut" class="com.example.learnspring.demo02.AnnotationPointCut"/>
<!--开启注解支持 JDK代理（默认）proxy-target-class="false" cglib proxy-target-class="true"-->
    <aop:aspectj-autoproxy proxy-target-class="true"/>
```

切面类：

```java
package com.example.learnspring.demo02;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect // 标注这是一个切面
public class AnnotationPointCut {

    @Before("execution(* com.example.learnspring.demo02.UserServiceImp.*(..))")
    public void before() {
        System.out.println("before: ======");
    }

    @After("execution(* com.example.learnspring.demo02.UserServiceImp.*(..))")
    public void after() {
        System.out.println("after: ======");
    }

    @Around("execution(* com.example.learnspring.demo02.UserServiceImp.*(..))")
    public void around(ProceedingJoinPoint jp) throws Throwable{
        System.out.println("before around:======");
        Signature signature = jp.getSignature();
        System.out.println("signature:" + signature); // 获得签名

        Object proceed = jp.proceed(); // 下一个执行方法或对象
        System.out.println("after around:======");

        System.out.println("proceed: " + proceed);
    }
}
```

执行结果（注意执行的顺序）：

```
before around:======
signature:void com.example.learnspring.demo02.UserService.add()
before: ======
add a user
after: ======
after around:======
proceed: null
```

