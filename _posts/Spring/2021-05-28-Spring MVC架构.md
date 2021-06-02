---
layout:     post
title:      "Spring - Spring MVC"
subtitle:   " \"Spring Knowledge - 11\""
date:       2021.05.28 12:29:00
author:     "Wuy"
header-img: "img/post-bg-2020.jpg"
catalog: true
tags:
    - 学习笔记
    - 读书
    - Spring
---

> *"Keep Learning Spring"*

# MVC的概念

MVC是一个架构，或者说是一个设计模式，它就是强制性使应用程序的输入，处理和输出分开。将一个应用程序分为三个部分：Model，View，Controller。

原理图：

![202105031012295911.png](https://i.loli.net/2021/05/28/l8kN4qP6hJOdbFD.png)

**Model模型**：负责完成业务逻辑：由JavaBean构成，在MVC的三个部件中，模型拥有最多的处理任务。例如它可能用像EJB和javabean这样的构件对象来处理数据库。由于应用于模型的代码只需写一次就可以被多个视图重用，所以减少了代码的重复性。

**View视图**：就是负责跟用户交互的界面。一般就是由HTML，css元素组成的界面，当然现在还有一些像js，ajax，flex一些也都属于视图层。 在视图层里没有真正的处理发生，之负责数据输出，并允许用户操纵的方式。MVC能为应用程序处理很多不同的视图。

**Controller控制器**：负责接收请求—>调用模型—>根据结果派发页面并经过模型处理返回相应数据。

# `DispatcherServlet`

首先看下继承关系：

![image-20210530105101762](https://i.loli.net/2021/05/30/gXlHMZEjT34UJeu.png)

执行流程如下：

![image-20210531104534023](https://i.loli.net/2021/05/31/nktv2aJ5YPryh6E.png)

组件说明：

- DispatcherServlet：前端控制器。用户请求到达前端控制器，它就相当于mvc模式中的c，dispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求，dispatcherServlet的存在降低了组件之间的耦合性,系统扩展性提高。由框架实现
- HandlerMapping：处理器映射器。HandlerMapping负责根据用户请求的url找到Handler即处理器，springmvc提供了不同的映射器实现不同的映射方式，根据一定的规则去查找,例如：xml配置方式，实现接口方式，注解方式等。由框架实现
- Handler：处理器。Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。由于Handler涉及到具体的用户业务请求，所以一般情况需要程序员根据业务需求开发Handler。
- HandlAdapter：处理器适配器。通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。由框架实现。
- ModelAndView是springmvc的封装对象，将model和view封装在一起。
- ViewResolver：视图解析器。ViewResolver负责将处理结果生成View视图，ViewResolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。
- View:是springmvc的封装对象，是一个接口, springmvc框架提供了很多的View视图类型，包括：jspview，pdfview,jstlView、freemarkerView、pdfView等。一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由程序员根据业务需求开发具体的页面。

测试类：

- 新建web项目，结构如下：

  ![image-20210531123053896](https://i.loli.net/2021/05/31/4aAudE95ZtDNQvn.png)

- 编写web.xml:

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
           version="4.0">
  
  <!--1. 注册DispatcherServlet-->
      <servlet>
          <servlet-name>springmvc</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <!--关联一个SpringMVC的配置文件-->
          <init-param>
              <param-name>contextConfigLocation</param-name>
              <param-value>classpath:springmvc-servlet.xml</param-value>
          </init-param>
  <!--启动级别-->
          <load-on-startup>1</load-on-startup>
      </servlet>
  <!--/ 匹配所有的请求，但不包括.jsp-->
  <!--/* 匹配所有的请求，包括.jsp-->
      <servlet-mapping>
          <servlet-name>springmvc</servlet-name>
          <url-pattern>/</url-pattern>
      </servlet-mapping>
  </web-app>
  ```

- 编写springmvc-servlet.xml:

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
     http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
     http://www.springframework.org/schema/context
     http://www.springframework.org/schema/context/spring-context-3.0.xsd">
  
      <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
      <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
  
      <!--使用注解开发时不需要配置映射器和适配器，只需要下面这三行用于扫描bean即可-->
      <!--context:component-scan base-package="com.lovecamille" /  使得指定包下的注解生效-->
      <!--mvc:default-servlet-handler/ 不处理静态文件--> 
      <!--mvc:annotation-driven/-->
  
  <!--视图解析器DispatcherServlet给它的ModelAndView-->
      <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
  <!--前缀-->
          <property name="prefix" value="/WEB-INF/jsp/" />
  <!--后缀-->
          <property name="suffix" value=".jsp" />
      </bean>
  
  </beans>
  ```

- 编写`Controller`:

  ```java
  package com.lovecamille.controller;
  
  
  import org.springframework.web.servlet.ModelAndView;
  import org.springframework.web.servlet.mvc.Controller;
  
  public class controller implements Controller {
  
      @Override
      public ModelAndView handleRequest(javax.servlet.http.HttpServletRequest httpServletRequest, javax.servlet.http.HttpServletResponse httpServletResponse) throws Exception {
          // ModelAndView 模型和视图
          ModelAndView mv = new ModelAndView();
  
          // 封装对象，放在ModelAndView中
          mv.addObject("msg", "Hello lovecamille");
  
          // 封装要跳转的视图，放在ModelAndView中
          mv.setViewName("hello");
          return mv;
      }
  }
  ```

  另一种注解开发的方式如下：

  ```java
  package com.lovecamille.controller;
  
  import org.springframework.stereotype.Controller;
  import org.springframework.ui.Model;
  import org.springframework.web.bind.annotation.RequestMapping;
  
  @Controller
  public class HelloController {
  
      @RequestMapping("/hello")
      public String printHello(Model model) {
  
          // 封装数据
          model.addAttribute("msg", "hello, lovecamille");
          // 会被
          return "hello";
      }
  
  
  }
  ```

- 编写前端界面：

  ```jsp
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
  <head>
      <title>
          hello
      </title>
  </head>
  <body>
  ${msg}
  </body>
  </html>
  ```
  
- 测试时发现使用Tomcat 10会找不到资源，具体报错如下：

  ![image-20210601100736851](https://i.loli.net/2021/06/01/IdohC8NHYna7JrK.png)

  也就是说当前Spring版本是不支持Tomcat 10的，所有的JavaEE的类都会被重命名以匹配JarkartaEE。解决办法就是降级为Tomcat 9. 一个Github Issue提到了这个问题：

  ![image-20210601101242125](https://i.loli.net/2021/06/01/3OlPruwKVJebipB.png)

  

# 转发与重定向

重定向是将用户从当前处理请求定向到另一个视图（例如 [JSP](http://c.biancheng.net/jsp/)）或处理请求，以前的请求（request）中存放的信息全部失效，并进入一个新的 request 作用域；转发是将用户对当前处理的请求转发给另一个视图或处理请求，以前的 request 中存放的信息不会失效。

转发是服务器行为，重定向是客户端行为。

## 转发过程

客户浏览器发送 http 请求，Web 服务器接受此请求，调用内部的一个方法在容器内部完成请求处理和转发动作，将目标资源发送给客户；在这里转发的路径必须是同一个 Web 容器下的 URL，其不能转向到其他的 Web 路径上，中间传递的是自己的容器内的 request。

在客户浏览器的地址栏中显示的仍然是其第一次访问的路径，也就是说客户是感觉不到服务器做了转发的。转发行为是浏览器只做了一次访问请求。

## 重定向过程

客户浏览器发送 http 请求，Web 服务器接受后发送 302 状态码响应及对应新的 location 给客户浏览器，客户浏览器发现是 302 响应，则自动再发送一个新的 http 请求，请求 URL 是新的 location 地址，服务器根据此请求寻找资源并发送给客户。

在这里 location 可以重定向到任意 URL，既然是浏览器重新发出了请求，那么就没有什么 request 传递的概念了。在客户浏览器的地址栏中显示的是其重定向的路径，客户可以观察到地址的变化。重定向行为是浏览器做了至少两次的访问请求。

示例如下：

```java
package com.lovecamille.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/index")
public class RedirectController {

    @RequestMapping("/isLogin")
    public String login(Model model) {
        model.addAttribute("msg", "forward");
        return "forward:/WEB-INF/jsp/hello.jsp";
    }

    @RequestMapping("/isAdmin")
    public String admin(Model model) {
        return "redirect:/WEB-INF/jsp/hello.jsp";
    }
}

```

# 接受请求参数及数据回显

具体看下示例就可以，这部分比较简单，也就是可以直接传递对象实例：

```java
package com.lovecamille.controller;

import com.lovecamille.pojo.User;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class ParameterController {

    // http://localhost:8080/springmvc_02_annotation_war_exploded/t1?name=XXX
    @GetMapping("/t1")
    public String test1(@RequestParam("name") String name, Model model) {

        // 1. receive parameter
        System.out.println(name);

        // 2. return parameter to front end
        model.addAttribute("msg", name);

        // 3. direct to specific page
        return "hello";
    }

    // http://localhost:8080/springmvc_02_annotation_war_exploded/t2?id=1&name=wuyan&age=3
    // 这里注意如果勇RestFul风格开发的话就类似于使用@RequestMapping加Method参数或者直接GetMapping
    @GetMapping("/t2")
    public String test2(User user) {
        System.out.println(user);
        return "hello";
    }
}
```

# 乱码问题

- 修改前端界面:

  ```jsp
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
      <head>
          <title>Test</title>
      </head>
      <body>
      <form action="" method="post">
          <input type="text" name="name">
          <input type="submit">
      </form>
      </body>
  </html>
  ```

  

- 测试类：

  ```java
  package com.lovecamille.controller;
  
  import org.springframework.stereotype.Controller;
  import org.springframework.ui.Model;
  import org.springframework.web.bind.annotation.PostMapping;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.RequestMethod;
  
  @Controller
  public class FilterController {
  
      // 注意这里如果使用浏览器测试的话，输入url默认是使用Get方法的，所以这里不能改成@PostMapping
      @RequestMapping(value = "/encoding")
      public String test1(String name, Model model) {
  
          model.addAttribute("msg", name);
          return "hello";
      }
  }
  ```

- 输出乱码（中文乱码）：

  ![image-20210601123110922](https://i.loli.net/2021/06/01/PvbBnlzUoF7YLZJ.png)

- 在web.xml中添加Spring的过滤器配置：

  ```xml
  <filter>
      <filter-name>filter</filter-name>
      <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
      <init-param>
          <param-name>encoding</param-name>
          <param-value>utf-8</param-value>
      </init-param>
  </filter>
  
  <filter-mapping>
      <filter-name>filter</filter-name>
      <url-pattern>/*</url-pattern>
  </filter-mapping>
  ```

  注意URL Pattern一定要是带星号的，保证可以定位到JSP文件。

# Jackson的使用

- 原始情况下，使用以下方式返回一个Json字符串：

  ```java
  package com.lovecamille.controller;
  
  import com.fasterxml.jackson.core.JsonProcessingException;
  import com.fasterxml.jackson.databind.ObjectMapper;
  import com.lovecamille.pojo.User;
  import org.springframework.stereotype.Controller;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.ResponseBody;
  
  @Controller
  public class UserController {
  
      @RequestMapping(value = "/j1", produces = "application/json;charset=utf-8")
      @ResponseBody // 添加这个注解就不会走视图解析器，会直接返回一个字符串；另一种方式就是类前面直接加上@RestController注解，不需要后面的@ResponseBody
      public String json1() throws JsonProcessingException {
  
          // jackson, ObjectMapper
          ObjectMapper mapper = new ObjectMapper();
  
          // create an object
          User user = new User("wuyan", 1, "male");
  
          String str = mapper.writeValueAsString(user);
  
          return str;
      }
  }
  ```

- 也可以使用配置文件解决乱码问题，而不是加`produces`参数：

  ```xml
  <!--Jackson乱码解决-->
  <mvc:annotation-driven>
      <mvc:message-converters>
          <bean class="org.springframework.http.converter.StringHttpMessageConverter">
              <constructor-arg value="UTF-8"/>
          </bean>
          <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
              <property name="objectMapper">
                  <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                      <property name="failOnEmptyBeans" value="false"/>
                  </bean>
              </property>
          </bean>
      </mvc:message-converters>
  </mvc:annotation-driven>
  ```

- 对时间的展示如下：

  ```java
      @RequestMapping("/j2")
      @ResponseBody
      public String json2() throws JsonProcessingException {
          ObjectMapper mapper = new ObjectMapper();
          
          // 不使用时间戳的方式，自定义时间格式
          mapper.configure(SerializationFeature.WRITE_DATE_KEYS_AS_TIMESTAMPS, false);
  
          SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
          mapper.setDateFormat(sdf);
  
          Date date = new Date();
          // return sdf.format(date);
          return mapper.writeValueAsString(date);
      }
  ```

  
