---
layout:     post
title:      "Spring - IOC本质"
subtitle:   " \"Spring Knowledge - 01\""
date:       2021.01.17 12:29:00
author:     "Wuy"
header-img: "img/post-bg-2020.jpg"
catalog: true
tags:
    - 学习笔记
    - 读书
    - Spring

---

> *"Keep Learning Spring"*

# Spring

## IOC本质

控制反转是一种设计思想，DI（依赖注入）是实现IOC的一种方法，也有人认为DI只是IOC的另一种说法。没有IOC的程序中，我们使用面向对象编程，对象的创建与对象间的依赖关系完全硬编码在程序中，对象的创建由程序自己控制，控制反转后将对象的创建转移给第三方，个人认为所谓控制反转就是：获得依赖对象的方式反转了。

IOC是Spring框架的核心内容，使用多种方式完美地实现了IOC，可以使用XML配置，也可以使用注解，新版本的Spring也可以零配置实现IOC。

Spring容器在初始化的时候先读取配置文件，根据配置文件或元数据创建与组织对象存入容器，程序使用时再从IOC容器中取出需要的对象。

采用XML方式配置Bean的时候，Bean的定义信息是和实现分离的，而采用注解的方式可以把两者合为一体，Bean的定义信息直接以注解的形式定义在实现类中，从而达到了零配置的目的。

**控制反转是一种通过描述（XML或注解）并通过第三方去生产或获取特定的对象的方式，在Spring中实现控制反转的是IOC容器，其实现方法是依赖注入（Dependency Injection， DI）。**

## Hello Spring

1. 撰写配置文件：

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <!-- using Spring to create objects, they are all called Bean in Spring 
       
       id = variables name
       class = object's absolute path
       property = set a value for one of object's properties
       
       -->
       <bean id = "hello" class="com.lov3camille.springtest.pojo.Hello">
           <property name="str" value="Spring"/>
   
       </bean>
   
   </beans>
   ```

2. `Hello.java`

   ```java
   package com.lov3camille.springtest.pojo;
   
   public class Hello {
   
       String str;
   
       public void setStr(String str) {
           this.str = str;
       }
   
       public String getStr() {
           return str;
       }
   
       @Override
       public String toString() {
           return "Hello" + str;
       }
   }
   
   ```

   

3. `MyTest.java`

   ```java
   package com.lov3camille.springtest;
   
   import com.lov3camille.springtest.pojo.Hello;
   import org.springframework.context.ApplicationContext;
   import org.springframework.context.support.ClassPathXmlApplicationContext;
   
   public class MyTest {
   
       public static void main(String[] args) {
   
           // resolve applicationContext.xml file and generate/manage Bean object
           ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
           // getBean()'s parameter is id of bean in .xml file
           Hello hello = (Hello) context.getBean("hello");
   
           System.out.println(hello.toString());
       }
   }
   ```

4. 几个关键点：

   - `Hello`对象是Spring创建的；
   - `Hello`对象的属性是由Spring容器设置的。
   - 上面几个过程就叫做控制反转：
     - 控制：传统的应用程序的对象是程序本身创建的，使用Spring后，对象由后者创建；
     - 反转：程序本身不创建对象，而变成被动地接收对象；
     - 依赖注入：就是利用`set`方法来进行注入；
   - `XML`文件中`ref`指引用容器中创建好的对象，`value`指具体的值，一般是基本数据类型；

## IOC创建对象的方式

1. 使用无参构造创建对象，这是默认的实现；

2. 假设我们需要使用有参构造：

   - ```xml
     <bean id = "hello" class="com.lov3camille.springtest.pojo.Hello">
         <constructor-arg index="0" value="wuyantest"/>
     </bean> 
     <!-- 下标赋值 -->
     ```

   - ```xml
     <bean id="user" class="com.lov3camille.springtest.pojo.Hello">
     	<constructor-arg type="java.lang.String" value="wuyan"/>
     </bean>
     <!-- 通过类型创建，但是不建议使用 -->
     ```

   - ```xml
     <bean id="user" class="com.lov3camille.springtest.pojo.Hello">
     	<constructor-arg name="name" value="wuyan"/>
     </bean>
     <!-- 直接通过参数名来设置 -->
     ```

3. 注入Bean的时候，尽管没有显式地注入，Bean里的对象都会被实例化并输出（如果有构造器的话），而且只会有一个实例。















































