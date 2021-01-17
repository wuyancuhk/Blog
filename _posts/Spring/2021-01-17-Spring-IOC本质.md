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