---
layout:     post
title:      "Spring - 使用JavaConfig实现配置"
subtitle:   " \"Spring Knowledge - 05\""
date:       2021.02.22 12:29:00
author:     "Wuy"
header-img: "img/post-bg-2020.jpg"
catalog: true
tags:
    - 学习笔记
    - 读书
    - Spring


---

> *"Keep Learning Spring"*

# 使用JavaConfig实现配置

1. 配置文件：

   ```java
   package config;
   
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.ComponentScan;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.context.annotation.Import;
   import pojo.User;
   
   // 这个也会被Spring容器托管，注册到容器中，因为它本来就是一个@Component
   // @Configuration代表这是一个配置表，类似于之前的beans.xml
   @Configuration
   @ComponentScan("pojo")
   @Import(FakerConfigTwo.class)
   public class FakerConfig {
   
       // 这代表注册一个bean，就相当于之前的一个bean标签，其中：
       // 这个方法的名字，相当于bean标签中的id；
       // 这个方法的返回值，相当于bean标签中的class属性。
       @Bean(name = "wuyan")
       public User getUser() {
           return new User(); // 这就是返回要注入到bean的对象。
       }
   
   }
   ```

2. 构建一个空的配置文件叫做`FakerConfigTwo.java`;

3. 构建用户类：

   ```java
   package pojo;
   
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.stereotype.Component;
   
   // 这个注解的意思就是，这个类被Spring接管了，注册到了容器中。
   @Component
   public class User {
   
       private String name;
   
       public String getName() {
           return name;
       }
   
       @Value("wuyan") // 属性注入值
       public void setName(String name) {
           this.name = name;
       }
   
       @Override
       public String toString() {
           return "User{" +
                   "name='" + name + '\'' +
                   '}';
       }
   }
   ```

4. 构建测试类：

   ```java
   import config.FakerConfig;
   import org.springframework.context.ApplicationContext;
   import org.springframework.context.annotation.AnnotationConfigApplicationContext;
   import pojo.User;
   
   public class MyTest {
       public static void main(String[] args) {
           ApplicationContext context = new AnnotationConfigApplicationContext(FakerConfig.class);
           User getUser = (User) context.getBean("wuyan");
           System.out.println(getUser.getName());
       }
   }
   ```

5. 最后输出为：

   ```
   23:19:48.942 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@3d04a311
   23:19:48.962 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor'
   23:19:49.102 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerProcessor'
   23:19:49.102 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerFactory'
   23:19:49.102 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalAutowiredAnnotationProcessor'
   23:19:49.112 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalCommonAnnotationProcessor'
   23:19:49.112 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'fakerConfig'
   23:19:49.122 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'config.FakerConfigTwo'
   23:19:49.122 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'wuyan'
   wuyan
   ```

   