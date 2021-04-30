---
layout:     post
title:      "Core Java - Daily Notes - 09"
subtitle:   " \"JAVA Knowledge - Advanced Features\""
date:       2020.09.11 15:29:00
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

### 4.4. 条件对象

通常，线程进入临界区后却发现只有满足了某个条件后它才能执行，可以使用一个**条件对象**来管理那些已经获得了一个锁却不能做有用工作的线程。

仍然是前面所述的例子，我们可以首先添加循环条件，等待其他线程向账户增加资金，然而由于锁的排他性访问权，别的线程没有存款机会，那么怎么办呢？

这时候需要引入条件对象，并且为它定义一个合适的名字，比如：

```java
class Bank
{
	private Condition sufficiendFunds;
	...
	public Bank()
	{
		...
		sufficientFunds = bankLock.newCondition();
	}
}
```

如果资金不足，那么它会调用`sufficientFunds.wait();`，当前线程现在就会暂停并放弃锁。

注意，这里等待获得锁的线程和已经调用了`await`方法的线程存在本质上的不同，一旦一个线程调用了`await`方法，它就进入了这个条件的**等待集**。当锁可用时，该线程并不会变为可运行状态，实际上，它仍处于非活动状态，直到另一个线程在同意条件上调用`signalAll`方法。注意，这一方法也只是说，现在有可能满足条件，值得再次检查条件。

如果没有其他线程来重新激活等待的线程，那么这一线程就不再运行了，这就是**死锁**现象，当然，最后一个线程如果还是不满足的话，程序就会永远挂起。

另一个方法`singnal`只是随机选择等待集中的一个线程，并解除这个线程的阻塞状态，虽然高效但也存在风险，原因如上所述。