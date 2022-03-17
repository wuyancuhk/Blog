---
layout:     post
title:      "SpringBoot - SpringBoot入门"
subtitle:   " \"SpringBoot Knowledge - 1\""
date:       2021.06.03 12:29:00
author:     "Wuy"
catalog: true
tags:
    - 学习笔记
    - SpringBoot

---

> *"Keep Learning SpringBoot"*

# SpringBoot基本配置

## Yaml语法

```yaml
# normal key-value
name: wuyan

#Object
student:
  name: wuyan
  age: 3
  
# inline style
student: {name: wuyan, age: 3}

# array
pets:
  - cat
  - dog
  - pig
    
pets: [cat, dog, pig]
```

相比于`*.properties` 文件，语法更为灵活。

### 示例

- 配置依赖：

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-configuration-processor</artifactId>
  </dependency>
  ```

- 创建一个`Person`类：

  ```java
  package com.example.learnspringboot.pojo;
  
  import java.util.List;
  import java.util.Map;
  import lombok.AllArgsConstructor;
  import lombok.Data;
  import lombok.NoArgsConstructor;
  import org.springframework.boot.context.properties.ConfigurationProperties;
  import org.springframework.stereotype.Component;
  
  @Data
  @NoArgsConstructor
  @AllArgsConstructor
  @Component
  @ConfigurationProperties(prefix = "person")
  public class Person {
      private int age;
      private String name;
      private Map<String, Object> map;
      private List<Object> list;
  }
  ```

- 配置Yaml文件：

  ```yaml
  person:
    age: 3
    name: wuyan
    map: {k1:v1, k2:v2}
    list:
      - dog
      - cat
      - wsm
  ```

- 编写测试类：

  ```java
  package com.example.learnspringboot;
  
  import com.example.learnspringboot.pojo.Person;
  import org.junit.jupiter.api.Test;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.boot.test.context.SpringBootTest;
  
  @SpringBootTest
  class LearnSpringBootApplicationTests {
  
      @Autowired
      private Person person;
  
      @Test
      void contextLoads() {
          System.out.println(person);
      }
  
  }
  ```

- 输出结果

  ```
  Person(age=3, name=wuyan, map={k1v1=, k2v2=}, list=[dog, cat, wsm])
  ```

  当然，也可以通过加`@Value`注解的方式配置属性。或者使用`@PropertySource("classpath:application.properties")`注解配合SPEL表达式（例如`@Value(${name})`）来指定配置文件。

- 多环境配置（SpringBoot 2.4 之后的配置与以往不同，示例如下）：

  ```yaml
  spring:
    config:
      activate:
        on-profile: test
  server:
    port: 8080
    
  ---
  spring:
    config:
      activate:
        on-profile: prod
  server:
    port: 8081
  ```


## MVC配置原理

关于这个问题，我们可以手写一个自己的配置类，然后对`DispacherServlet`类打上断点进行调试分析：

- `MyMVCConfig`

  ```java
  package com.example.learnspringboot.config;
  
  import java.util.Locale;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.web.servlet.View;
  import org.springframework.web.servlet.ViewResolver;
  import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
  
  @Configuration
  public class MyMVCConfig implements WebMvcConfigurer {
  
      // if a class implements viewResolver interface, then it can be called a view resolver
      @Bean
      public ViewResolver myViewResolver() {
          return new MyViewResolver();
      }
  
      public static class MyViewResolver implements ViewResolver {
          @Override
          public View resolveViewName(String s, Locale locale) throws Exception {
              return null;
          }
      }
  
  }
  ```

- ![image-20210606110702245](https://i.loli.net/2021/06/06/MDSxmu1bWrYk5J8.png)

- 可以发现自己定制的`viewResolver`已经加入到了列表中：

  ![image-20210606110817330](https://i.loli.net/2021/06/06/ePRm7Q5AFXvy2xE.png)

  

# 简易SpringBoot管理系统

- 项目结构大致如下：

	![image-20210609101703814](https://i.loli.net/2021/06/09/kCxJjEeG8bd4XoV.png)
	
- GitHub地址：

  https://github.com/wuyancuhk/employeeManager.git





