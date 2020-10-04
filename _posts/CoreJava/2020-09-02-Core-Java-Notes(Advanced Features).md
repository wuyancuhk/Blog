---
layout:     post
title:      "Core Java - Daily Notes - 05"
subtitle:   " \"JAVA Knowledge - Advanced Features\""
date:       2020.09.02 14:29:00
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

## 5. 日志

有问题的代码可以插入一些`System.out.prinln`方法来帮助观察程序行为，但是这样做比较繁琐。日志`API`就是为了解决这个问题而设计的。在Java 9中，Java平台有一个单独的轻量级日志系统，比Log4J 2或Logback之类的要略逊一筹，但是正常开发应用程序也都用不到。

### 5.1. 基本日志

要生成简单的日志记录，可以使用全局日志记录器并调用其`info`方法：

```java
Logger.getGlobal().info("File -> Open menu item selected");
```

在默认情况下，会打印这个记录：

```java
9月 02, 2020 3:59:13 下午 DailyTest.Main main
信息：File -> Open menu item selected
```

但是，如果在适当的地方（如`main`的最前面）调用：

```java
Logger.getGlobal().setLevel(Level.OFF);
```

### 5.2. 高级日志

在一个专业的应用程序中，肯定不希望所有的日志都记录到一个全局日志记录器中，我们可以定义自己的日志记录器（创建或者获取）：

```java
private static final Logger myLogger = Logger.getLogger("com.mycompany.myapp")
```

注意，未被任何变量引用的日志记录器可能会被垃圾回收，为了防止这种情况，要用静态变量存储日志记录器的引用。

与包名类似，日志记录器名也有层次结构，并且更强。包与父包之间没有语义关系，但实日志记录器则会共享某些属性。比如日志级别。

而通常来讲，日志级别有七类：

- `SEVERE`
- `WARNING`
- `INFO`
- `CONFIG`
- `FINE`
- `FINER`
- `FINEST`

在默认情况下，实际上只记录前三个级别，也可以设置一个不同的级别，例如：

```java
logger.setLevel(Level.FINE);
```

现在，`FINE`以及所有更高级别的日志都会记录。

所有级别都有日志记录方法，如：

```java
logger.warning(message);
logger.fine(message);
```

或者，还可以使用`log`方法并指定级别，例如：

```java
logger.log(Lever.FINE, message);
```

默认的日志记录将显示根据调用堆栈得出的包含日志调用的类名和方法名，不过，如果虚拟机对执行过程进行了优化，就得不到准确的调用信息，此时，可以使用`logp`方法获得调用类和方法的确切位置，这个方法的示例如下:

```java
log.logp(Level.WARNING, String.valueOf(Main.class), "factorial", "test" );
```

当然，记录日志的常见用途是记录那些预料之外的异常，可以使用如下两个便利的方法：

```java
if (...)
{
    var e = new IOException("...");
    logger.throwing("DailyTest.Main", "read", e);//format: void throwing(String className, String methodName, Throwable t)
    throw e;
}
```

或者：

```java
try
{
	...
}
catch (IOException e)
{
	Logger.getLogger("DailyTest.Main").log(Level.WARNING, "Reading Message", e);//format: void log(Level l, String message, Throwable t)
}
```

### 5.3. 日志技巧

下面总结以下日志的使用技巧：

1. 对一个简单的应用，选择一个日志记录器，并且可以为有大量日志记录活动的类增加静态字段；
2. 默认的日志配置会把级别等于或高于`INFO`的所有消息记录到控制台，用户可以覆盖这个默认配置；
3. 要牢记：所有级别为`INFO`、`WARNING`、`SEVERE`的消息都将显示在控制台上，所以，最好将对用户有意义的消息设置为这几个级别，将想要的日志消息设定为`FINE`级别是一个很好的选择。