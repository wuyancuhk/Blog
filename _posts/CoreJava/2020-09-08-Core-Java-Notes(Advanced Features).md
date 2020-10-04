---
layout:     post
title:      "Core Java - Daily Notes - 07"
subtitle:   " \"JAVA Knowledge - Advanced Features\""
date:       2020.09.08 15:29:00
author:     "Wuy"
header-img: "img/post-bg-2020.jpg"
catalog: true
tags:
    - 学习笔记
    - 读书
    - JAVA


---

> *"Keep Learning JAVA"*

# 并发

### 3.2. 守护线程

可以通过调用`t.setDaemon(true);`将一个线程转换为守护线程，它的唯一用途就是为其他线程提供服务，比如计时器线程和清空果实缓存项的线程。注意这一方法必须在线程启动之前调用。

### 3.3. 线程名

默认情况下，线程名字是`Thread-number`，可以用`setName`方法为线程设置任何名字：

```java
var t = new Thread(runnable);
t.setName("Web crawler");
```

### 3.4. 未捕获异常的处理器

线程的`run`方法不能抛出任何检查型异常，但是，非检查型异常可能会导致线程终止。在这种情况下，线程会死亡。

不过，对于可传播的异常，并没有任何`catch`子句。实际上，在线程死亡之前，异常会传递到一个用于处理未捕获异常的处理器。这个处理器必须是一个实现了`Thread.UncaughtExceptionHandler`接口的类，这个接口只有一个方法：

```java
void uncaughtException(Thread t, Throwable e);
```

可以用`setUncaughtExceptionHandler`方法为任何线程安装一个处理器。也可以用`Thread`类的静态方法`setDefaultUncaughtExceptionHandler`为所有线程安装一个默认的处理器。替代处理器可以使用日志API将未捕获异常的报告发送到一个日志文件。

如果没有安装默认处理器，那么默认处理器则为`null`,但是，如果没有为单个线程安装处理器，那么处理器就是该线程的`ThreadGroup`对象。（建议不要在自己的程序里使用线程组）

`ThreadGroup`类实现了上述接口，它的方法执行以下操作：

1. 如果该线程组有父线程组。那么调用父线程组的`uncaughtException`方法。
2. 否则，如果`Thread.getDefaultExceptionHandler`方法返回一个非`null`的处理器，则调用该处理器。
3. 否则，如果`Throwable`是`ThreadDeath`的一个实例，什么都不做。
4. 否则，将线程的名字以及`Throwable`的栈轨迹输出到`System.err`。

### 3.5. 线程优先级

线程调度机会优先选择较高优先级的线程，但是线程优先级高度依赖于系统，Java线程的优先级会映射到宿主机平台的优先级，所以现在基本不使用线程优先级了。