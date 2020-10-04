---
layout:     post
title:      "Core Java - Daily Notes - 08"
subtitle:   " \"JAVA Knowledge - Advanced Features\""
date:       2020.09.10 15:29:00
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

## 4. 同步

如果两个线程存取同一个对象，并且每个线程分别调用了一个修改该对象状态的方法，两个线程会相互覆盖，会导致对象被破坏，这种情况通常称为**竞态条件**。

### 4.1. 竞态条件的一个例子

为了避免多线程破坏共享数据，必须学习**同步存取**，下面会以一个例子先介绍没有同步存取会发生什么：

```java
public class DailyTest
{
    public static final int NACCOUNTS = 100;
    public static final double INITIAL_BALANCE = 1000;
    public static final double MAX_AMOUNT = 1000;
    public static final int DELAY = 10;

    public static void main(String[] args)
    {
        var bank = new Bank(NACCOUNTS, INITIAL_BALANCE);
        for (int i = 0; i < NACCOUNTS; i++)
        {
            int fromAccount = i;
            Runnable r = () -> {
                try
                {
                    while (true)
                    {
                        int toAccount = (int) (bank.size() * Math.random());
                        double amount = MAX_AMOUNT * Math.random();
                        bank.transfer(fromAccount, toAccount, amount);
                        Thread.sleep((int) (DELAY * Math.random()));
                    }
                }
                catch (InterruptedException e)
                {
                }
            };
            var t = new Thread(r);
            t.run();
        }
    }
}
```

这段程序在刚开始运行时总金额没问题，还是100000，但是运行一段时间后就出现了微笑的变化：

![image-20200910173927897](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\image-20200910173927897.png)

下面介绍出现这一问题的原因。

### 4.2. 竞态条件详解

当两个线程试图同时更新同一个账户时，就会出现这个问题，假设两个线程同时执行指令：

```java
accounts[to] += amount;
```

问题在于这不是原子操作。这个指令可能如下处理：

1. 将`accounts[to]`加载到寄存器；
2. 增加`amount`；
3. 将结果写回`accounts[to]`。

现在，假设第一个线程执行步骤1和2，然后，它的运行权被抢占，再假设第个线程被唤醒，更新`account`数组中的同一个元素，然后第1个线程被唤醒并完成其第3步。

这个动作会抹除第2个线程所作的更新。这样一来，总金额就不再正确了。

在一个有多核的现代处理器上，出问题的风险相当的高。虽然调度器不太可能在线程的计算过程中抢占它的运行权，但是，仍然有风险产生破坏。

### 4.3. 锁对象

有两种机制可防止并发访问代码块。也就是`synchronized`关键字。它会自动提供一个锁以及相关的“条件”。另外还有一个`ReentrantLock`类。使用这个类保护代码的基本结构如下：

```java
myLock.lock(); // a ReentrantLock Object
try
{
	... // critical section
}
finally
{
	myLock.unlock(); // make sure the lock is unlocked even if an exception is thrown, otherwise, other threads will be blocked forever
}
```

这个结构确保任何时刻只有一个线程进入临界区。一旦一个线程锁定了锁对象，其他任何线程都无法通过`lock`语句。当其他线程调用`lock`时，它们会暂停，直到第一个线程释放这个锁对象。

**注意，这里不能使用`try-with-resource`语句，因为它的首部是要希望声明一个新变量，但是使用锁意味着要使用多个线程共享的那个变量。**

下面使用一个锁来保护`Bank`类的`transfer`方法：

```java
public class Bank
{
    private var bankLock = new ReentrantLock();
    ...
    public void transfer(int from, int to, int amount)
    {
        bankLock.lock();
        try
        {
            System.out.println(Thread.currentThread());
            accounts[from] -= amount;
            System.out.printf("%10.2f from %d to %d", amount, from, to);
            accounts[to] += amount;
            System.out.printf(" Total Balance: %10.2f%n", getTotalBalance());
        }
        finally
        {
            bankLock.unlock();
        }
    }
}
```

每个对象都有自己的`ReentrantLock`对象，如果两个线程访问同一个对象，锁可以保证串行化访问，对于不同的对象，就会有不同的锁。

这个锁称为重入锁，因为线程可以反复获得已拥有的锁，它还有一个持有技术来跟踪对`lock`方法的嵌套调用。

同时还要确保临界区的代码不会因为抛出异常而跳出临界区。

另外，可以在`ReetrantLock()`方法中添加`boolean`参数，如果设为`true`，意味着这是一个公平锁，一个公平锁倾向于等待时间最长的线程，但是可能会严重影响线程。



