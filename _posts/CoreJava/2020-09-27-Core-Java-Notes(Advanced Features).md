---
layout:     post
title:      "Core Java - Daily Notes - 10"
subtitle:   " \"JAVA Knowledge - Advanced Features\""
date:       2020.09.27 15:29:00
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

### 4.5. `synchronized`关键字

先对锁和条件的要点做一个总结：

- 锁用来保护代码片段，一次只能有一个线程执行被保护的代码。
- 锁可以管理试图进入被保护代码段的线程。
- 一个锁可以有一个或者多个相关联的条件对象。
- 每个条件对象管理哪些已经进入被保护代码段但还不能运行的线程。

当然，从`Java 1.0`版本开始，如果一个方法声明时有`synchronized`关键字，那么对象的锁将保护整个方法。也就是说，要调用这个方法，线程必须获得内部对象锁。

换句话说，

```java
public synchronized void method()
{
	method body
}
```

等价于：

```java
public void method()
{
	this.intrinsicLock.lock();
	try
	{
		method body
	}
	finally
	{
		this.intrinsicLock.unlock();
	}
}
```

例如，可以简单地将`Bank`类地`transfer`方法声明为`synchronized`，而不必使用一个显示的锁。

内部对象锁只有一个关联条件。`wait`方法等价于`await()`,都是将一个线程增加到等待集中，`notifyAll/notify`方法可以解除等待线程的阻塞，等价于`signalAll()`方法。之所以命名不同，是因为上述提到的前一个方法都属于`Object`类的`final`方法，后者则是`Condition`方法，用来避免冲突。

内部所和条件存在一些限制，包括：

- 不能中断一个正在尝试获得锁的线程。
- 不能指定尝试获得锁的超时时间。
- 每个锁仅有一个条件可能是不够的。

要注意，以上的这些方法都只能在一个同步方法或同步块中调用，否则会抛出`IllegalMonitorStateException`异常。

### 4.6. 同步块

正如刚才提到的，我们有另一种机制可以获得锁：就是进入**同步块**。

在这里，创建`lock`对象只是为了使用每个Java对象拥有的锁。

有时程序员使用一个对象的锁来实现额外的原子操作，这种做法称为**客户端锁定**，但是，这完全依赖于一个事实：该类会对自己的所有更改方法使用内部锁，因此，这是非常脆弱的，不推荐使用。

### 4.7. 监视器概念

如何做到不要求考虑显式锁就可以保证多线程的安全性呢，解决方案之一是监视器。监视器具有如下特性：

- 监视器是只包含私有字段的类。
- 监视器类的每个对象有一个关联的类。
- 所有方法由这个锁锁定。
- 锁可以有任意多个相关联的条件。

然而，Java对象有三个方面不同于监视器，这也就削弱了线程的安全性：

- 字段不要求是`private`。
- 方法不要求是`synchronized`。
- 内部锁对客户是可用的。

总的来说，监视器是由Per Brich Hansen和Tony Hoare提出的概念，Java以不精确的方式采用了它，也就是Java中的每个对象有一个内部的锁和内部条件。如果一个方法用`synchronized`关键字声明，那么，它表现的就像一个监视器方法。通过`wait/notifyAll/nofify`来访问条件变量。

### 4.8. `volatile`字段

该关键字为实例字段的同步访问提供了一种免锁机制，如果声明一个字段为`volatile`，那么编译器和虚拟机就知道该字段可能被另一个线程并发更新。

或许使用内部对象锁并不是一个好主意，因为如果另一个线程已经对该对象加锁，那么当前方法就可能会阻塞，这时使用`volatile`字段就会让编译器插入适当的代码，以确保如果一个线程对变量做了修改，这个修改对读取这个变量的所有其他线程都可见。

**注意：**`volatile`关键字不能保证原子性。