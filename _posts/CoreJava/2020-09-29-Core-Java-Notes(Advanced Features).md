---
layout:     post
title:      "Core Java - Daily Notes - 11"
subtitle:   " \"JAVA Knowledge - Advanced Features\""
date:       2020.09.29 12:29:00
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

### 4.9. `final`变量

还有一种情况可以安全地访问一个共享字段，即这个字段声明为`final`时，考虑以下声明：

```java
final var accounts = new HashMap<String, Double>();
```

其他线程会在构造器完成构造后才看到这个`accounts`变量。但这也不是线程安全的，如果有多个线程读取和更改它，那么仍然需要同步，这里只是构造的时候是安全的。

### 4.10. 原子性

假设对共享变量除了赋值以外不做任何操作，那么可以将这些变量声明为`volatile`.

比如说`AtomicInteger`类的几个方法就以原子方式将一个整数进行自增或者自减，例如，可以安全地生成一个数值序列，并且该操作不会中断：

```java
public static AtomicLong nextNumber = new AtomicLong();

long id = nextNumber.incrementAndGet();
```

如果需要完成复杂的更新，比如追踪并更新最大值，使用`set()`方法是不行的，要使用原子性方法：

```java
import java.util.concurrent.atomic.AtomicLong;

public class Test
{
    public static AtomicLong largest = new AtomicLong();
    public static void main(String[] args)
    {
        long observed = 1000;
        largest.accumulateAndGet(observed, Math::max);
        System.out.println(largest);
    }
}
```

如果有大量线程要访问相同的原子值，性能就会大幅下降，因为乐观更新需要太多次重试了。通常情况下，只有当所有工作都完成后才需要总和的值，对于这种情况，这种方法会很高效。也就是说，如果预期存在大量竞争，只需要用`LongAdder`。

对于`LongerAdd`类的`increment()`方法，不会返回原值，因为这样会消除将求和分解到多个加数所带来的性能提升。

### 4.11. 死锁

有可能会因为每一个线程都要等待满足条件才能进行导致所有线程都被阻塞，这样的状态叫做死锁。

还有一种情况，那就是所有线程都集中到一个变量上的时候，会很快导致死锁。

最后一种情况就是，`singnalAll`方法修改为`singnal`方法，程序最终就会挂起，因为后者只会解除一个线程的阻塞，如果该线程不能运行，所有线程都会阻塞。

### 4.12. 线程局部变量

有时候可能要避免共享变量，使用`ThreadLocal`辅助类为各个线程提供各自的实例。 

比如要得到当前的日期，那么首先要为每个线程构造一个实例：

```java
public static final ThreadLocal<SimpleDateFormat> dateformat = ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));
```

要访问具体的格式化方法，就要调用：

```java
String dateStamp = dateformat.get().format(new Date());
```

在一个给定的线程中首次调用`get`时，会调用构造器中的`lambda`表达式，此后，`get`方法会返回属于当前线程的那个实例。