---
layout:     post
title:      "Core Java - Daily Notes - 06"
subtitle:   " \"JAVA Knowledge - Advanced Features\""
date:       2020.09.04 17:29:00
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

多线程程序在更低一层扩展了多任务的概念：单个程序看起来在同时完成多个任务，每个任务在一个线程中执行，线程是控制线程的简称。如果一个程序可以同时运行多个线程，则称这个程序是多线程的。

多线程和多进程的本质区别主要在于每个进程都有自己的一整套变量，而线程则是共享数据。共享变量使得线程之间的通信比进程之间的通信更有效、更容易。

## 1. 什么是线程

可以通过一个例子去了解：

```java
public class DailyTest
{
    public static final int DELAY = 10;
    public static final int STEPS = 100;
    public static final double MAX_AMOUNT = 1000;

    public static void main(String[] args)
    {
        var bank = new Bank(4, 100000);
        Runnable task1 = () ->
        {
            try
            {
                for (int i = 0; i < STEPS; i++)
                {
                    double amount = MAX_AMOUNT * Math.random();
                    bank.transfer(0, 1, amount);
                    Thread.sleep((int) (DELAY * Math.random()));
                }
            }
            catch (InterruptedException e)
            {
            }
        };

        Runnable task2 = () ->
        {
            try
            {
                for (int i = 0; i < STEPS; i++)
                {
                    double amount = MAX_AMOUNT * Math.random();
                    bank.transfer(2, 3, amount);
                    Thread.sleep((int) (DELAY * Math.random()));
                }
            }
            catch (InterruptedException e)
            {
            }
        };

        new Thread(task1).start();
        new Thread(task2).start();
    }
}
```

另外，还可以通过建立`Thread`类的一个子类来定义线程，如下所示：

```java
class MyThread extends Thread
{
	public void run()
	{
		...
	}
}
```

然后可以构造这个子类的一个对象，并调用它的`start`方法。但是现在并不推荐，应当把要并行运行的任务与运行机制解耦合。如果有多个任务，为每个任务创建一个单独的线程开销会太大。实际上，可以使用一个**线程池**。

**警告：**不要调用`Thread`类或者`Runnable`对象的`run`方法。直接调用`run`方法只会在同一个线程中执行这个任务，而没有启动兴的线程。实际上，应当调用`Thread.start`方法，这会创建一个执行`run`方法的新线程。另外，必须覆盖`Runnable`接口里的`run()`方法。

## 2. 线程状态

线程可以有如下六种状态：

- `New`新建
- `Runnable`可运行
- `Blocked`阻塞
- `Waiting`等待
- `Timed waiting`计时等待
- `Terminated`终止

要确定一个线程的当前状态，只需要调用`getState`方法。

### 2.1. 新建线程

通过类似于`new Thread(r)`指令创建一个新线程时，这个线程处于新建状态，还没有开始运行，还有其他基础工作要做。

### 2.2. 可运行线程

一旦调用`start`方法，线程就处于**可**运行状态。注意，正在运行不属于一个单独的状态，它仍然属于可运行状态。

现在的所有桌面以及服务器操作系统都使用抢占式调度，但是，像手机这种的小型设备可能使用协作式调度。在这样的设备中，一个线程只有在调用`yield`方法（使当前正在执行的线程向另一个线程交出运行权，注意这是一个静态方法）或者被阻塞或等待时候才失去控制权。在有多个处理器的机器上，每一个处理器运行一个线程，可以有多个线程并行运行。当然，如果线程的数目多于处理器的数目，调度器还是需要分配时间片。

### 2.3. 阻塞和等待线程

当线程处于这两种状态时，它暂时是不活动的，不运行任何代码，并且消耗最小的资源，要由线程调度器来重新激活这个线程。

具体细节取决于它是怎样到达非活动状态的：

- 当一个线程试图获取一个内部的对象锁，而这个锁目前被其他线程占有，该线程就会被阻塞，当所有其他线程都释放了这个锁后，并且线程调度器允许该线程持有这个锁时，它将变成非阻塞状态；
- 当线程等待另一个线程通知调度器出现一个条件时，这个线程会进入等待状态，它和阻塞状态没太大区别；
- 有几个方法有超时参数，调用这些方法会让线程进入计时等待状态，这一状态会一直保持到超时期满或者接收到适当通知。

注意，新线程的运行需要有这几个条件：

1. 因超时期满或成功获得一个锁而重新激活；
2. 比当前运行线程的优先级更高。

### 2.4. 终止线程

线程会由于以下两个原因之一而终止：

- `run`方法正常退出，线程自然终止；
- 因为一个没有捕获的异常终止了`run`方法，使线程意外终止。

## 3. 线程属性

### 3.1. 中断线程

除了已经废弃的`stop`方法，没有办法可以强制终止线程，不过，`interrupt`方法可以用来请求终止一个线程。

当对一个线程调用`interrupt`方法时，就会设置线程的中断状态，这是每个线程都有的`boolean`标志，每个线程都应该时不时地检查这个标志，以判断线程是否被中断，判断方法如下：

```java
while (!Thread.currentThread().isinterrupted() && ...)
{
    ...
}
```

但是，如果线程被阻塞，就无法检查中断状态，这里就要引入`InterruptedException`异常。当在一个被`sleep`或`wait`调用阻塞的线程上调用`interrupt`方法时，哪个阻塞调用将被一个`InterruptionException`异常中断。（有些阻塞`I/O`调用不能被中断，对此要选择可中断的调用）

如果设置了中断状态，此时如果调用`sleep`方法，它不会休眠，实际上，他会清除中断状态并抛出`InterruptionException`。因此，如果循环调用的`sleep`方法，不要检测中断状态，而是应当捕获`InterruptionException`异常。

有两个非常类似的方法，`interrupted`和`isInterrupted`， 前者是一个静态方法，会清除该线程的中断状态，另一方面，后者是一个实例方法，可以用来检查是否有线程被中断，调用这个方法不会改变中断状态。

注意，发布的代码最好不要抑制`InterruptionException`异常，抛出它或者设置中断状态都行。