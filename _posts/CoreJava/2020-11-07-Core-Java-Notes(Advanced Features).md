---
layout:     post
title:      "Core Java - Daily Notes - 17"
subtitle:   " \"JAVA Knowledge - Advanced Features\""
date:       2020.11.07 12:29:00
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

## 6. 任务和线程池

### 6.4. `fork-join`框架

有些应用使用了大量线程，但其中大多数都是空闲的。比如一些应用可能对每个处理器内核分别使用一个线程，以完成计算密集型任务，如图像或视频处理。`fork-join`框架就是用来支持这一应用的。大致处理如下所示：

```java
if (problemsize < threshold)
	...
else
{
	//break problem into subproblems
	//comine the results
}
```

对于这种类似于`MapReduce`的框架，需要提供一个扩展`RecursiveTask<T>`的类（如果计算会生成一个类型为`T`的结果）或者提供一个扩展`RecursiveAction`的类（如果不生成任何结果）。再覆盖`compute`方法来生成并调用子任务，然后合并结果。示例如下：

```java
class Counter extends RecursiveTask<Integer>
{
	...
	protected Integer compute()
	{
        if (to - from < THRESHOLD)
        {
            //solve problem directly
        }
        else
        {
        	int mid = (from + to) / 2;
        	var first = new Counter(value, from, mid. filter);
        	var second = new Counter(values, mid, to, filter);
        	invokeAll(first, second);
        	return first.join() + second.join();
        }
	}
}
```

在这里，`invokeAll`方法接收到很多任务并且阻塞，直到这些任务全部完成。`join`方法将生成全部结果。我们对每个子任务应用`join`，并返回其总和。

完整的示例代码如下，主要功能为输出随机生成的一千万个浮点数里有多少大于0.5：

```java
import java.util.concurrent.*;
import java.util.function.*;

public class DailyTest
{
    public static void main(String[] args)
    {
        final int SIZE = 10000000;
        var numbers = new double[SIZE];
        for (int i = 0; i < SIZE; i++) numbers[i] = Math.random();
        var counter = new Counter(numbers, 0, numbers.length, x -> x > 0.5);
        var pool = new ForkJoinPool();
        pool.invoke(counter);
        System.out.println(counter.join());
    }
}

class Counter extends RecursiveTask<Integer>
{
    public static final int THRESHOLD = 1000;
    private double[] values;
    private int from;
    private int to;
    private DoublePredicate filter;

    public Counter(double[] values, int from, int to, DoublePredicate filter)
    {
        this.values = values;
        this.from = from;
        this.to = to;
        this.filter = filter;
    }

    protected Integer compute()
    {
        if (to - from < THRESHOLD)
        {
            int count = 0;
            for (int i = from; i < to; i++)
            {
                if (filter.test(values[i])) count++;
            }
            return count;
        }
        else
        {
            int mid = (from + to) / 2;
            var first = new Counter(values, from, mid, filter);
            var second = new Counter(values, mid, to, filter);
            invokeAll(first, second);
            return first.join() + second.join();
        }
    }
}

```

在后台，`fork-join`框架使用了一种有效的只能方法来平衡可用线程的工作负载，这种方法称为工作密取。每个工作线程都有一个双端队列来完成任务，一个工作线程将子任务压入其双端队列的队头（只有一个线程可以访问队头，所以不需要加锁）， 当一个工作线程空闲时，他会从另一个双端队列队尾“密取”一个任务。由于大的任务都在队尾，所以这种密取很少出现。