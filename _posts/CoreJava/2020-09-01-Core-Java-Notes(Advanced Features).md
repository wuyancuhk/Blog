---
layout:     post
title:      "Core Java - Daily Notes - 04"
subtitle:   " \"JAVA Knowledge - Advanced Features\""
date:       2020.09.01 15:00:00
author:     "Wuy"
header-img: "img/post-bg-2020.jpg"
catalog: true
tags:
    - 学习笔记
    - 读书
    - JAVA


---

> *"Keep Learning JAVA"*

# 异常、断言和日志

## 4. 使用断言

在一个具有自我保护能力的程序中，断言很常用。

### 4.1. 断言的概念

假设确信某个属性符合要求，并且代码的执行依赖于这个属性，例如，需要计算：

```java
double y = Math.sqrt(x);
```

你确信这里的x是一个非负数，原因是这个x是通过其他计算得到的，或者是某个方法的参数（方法的调用者只能提供正数输入）。

那么你的确可以提供异常检查，然而这种检查过多后，程序会变得很慢。

**断言机制**允许在测试期间向代码中插入一些检查，而在生产代码中会自动删除这些检查。

举个例子：

如果想断言x是一个非负数，只需要用下面的语句：

```java
assert x >= 0;
```

或者将x的实际值传递给`AssertionError`对象，以便日后显示（注意这一对象仅存储一个消息字符串，如果使用表达式的值，那么就会违背了断言机制的初衷）：

```java
assert x >= 0 : x;
```

### 4.2. 启用和禁用断言

在默认情况下，断言是禁用的。可以在运行程序时用`-enableassertions`或`-ea`选项启用断言: `java -enableassertions MyApp`

需要注意的是，不必重新编译程序来启用或者禁用断言，这一动作是**类加载器**的功能。禁用断言时，类加载器会去除断言代码，因此，不会降低程序运行的速度。

也可以在某个类或者整个包中启用断言，例如：`java -ea: MyClass -ea: com.mycompany.mylib.MyApp`

反之，也可以用选项`-disableassertions`或`-da`在某个特定类和包中禁用断言。主要因为有些类不是由类加载器加载的，而是直接由虚拟机加载的。

不过，启用和禁用所有断言的开关不能应用到哪些没有类加载器的“系统类”上。对于这些系统类，需要使用`-enablesystemassertions/-esa`开关启用断言。

### 4.3 . 使用断言完成参数检查

使用断言时，要记住下面几点：

1. 断言失败是致命的、不可恢复的错误；
2. 断言检查只是在开发和测试阶段打开。

### 4.4. 使用断言提供假设文档

这是一种替代使用注释来提供底层假设的文档的一种方法。



最后，关于`ClassLoader`包里的几个`API`的用法，示例如下：

```java
// Java program to demonstrate the example 
// of void setDefaultAssertionStatus () method of ClassLoader 
 
public class setDefaultAssertionStatusOfClassLoader {
 public static void main(String[] args) throws Exception {
 
  // Load a class
  Class cl = Class.forName("setDefaultAssertionStatusOfClassLoader");
 
  // It returns the ClassLoader associated with the
  // class Object
  ClassLoader loader = cl.getClassLoader();
 
  // Display loader
  System.out.println("loader Class: " + loader.getClass());
 
  // By using setDefaultAssertionStatus() method is to set the 
  //the default status 
  loader.setDefaultAssertionStatus(true);
 }
}
```

以上程序的输出为：

```java
loader Class: class jdk.internal.loader.ClassLoaders$AppClassLoader
```

