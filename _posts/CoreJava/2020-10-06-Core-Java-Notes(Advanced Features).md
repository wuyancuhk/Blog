---
layout:     post
title:      "Core Java - Daily Notes - 15"
subtitle:   " \"JAVA Knowledge - Advanced Features\""
date:       2020.10.06 12:29:00
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

构造一个新的线程开销有点大，因为这涉及于操作系统的交互。如果你的程序中创建了大量的生命期很短的线程，那么不应该把每个人物映射到一个单独的线程，而应该使用线程池。线程池中包含很多准备运行的线程，为线程池提供一个`Runnable`，就会有一个线程调用`run`方法。当`run`方法退出时，这个线程不会死亡，而是留在池中准备为下一个请求提供服务。

### 6.1. `Callable`与`Future`

可以把`Runnable`想象成一个没有参数和返回值的异步方法。

`Callable`与之类似，但是它有返回值，这一接口是个参数化的类型，并且只有一个方法`call`。类型参数是返回值的类型。

格式如下：

```java
public interface Callable<V>
{
	V call() throw Exception;
}
```

`Future`则保存异步计算的结果，该接口有如下方法：

```
V get();
V get(long timeout, TimeUnit unit)
void cancel(boolean, mayInterrupt)
boolean isCancelled()
boolean isDone()
```

执行`Callable`的一种方法是使用`FutureTask`，它实现了`Future`和`Runnable`接口，所以可以构造一个线程来运行这个任务：

```java
Callable<Integer> task = new Callable<Integer>() {
@Override
    public Integer call() throws Exception {
    	return 1;
    }
};
Future futuretask = new FutureTask<Integer>(task);
Thread t = new Thread(String.valueOf(futuretask));
t.start();
Integer result = task.get();
```

### 6.2. 执行器

执行器类有很多静态工厂方法，用来构造线程池：

- `newCachedThreadPool`
- `newFixedThreadPool`
- `newWorkStealingPool`
- `newSingleThreadPool`
- `newScheduleThreadPool`
- `newSingleThreadScheduleExecutor`

如果线程生命周期短，那么就要使用一个缓存线程池，为了得到最优运行速度，并发线程数要等于处理器内核数，那么就要使用一个固定线程池。可用下面的方法之一讲`Runnable`或者`Callable`对象提交给`ExecutorService`：

```
Future<T> submit(Callable<T> task)
Future<T> submit(Runnable task)
Future<T> submit(Runnable task, T result)
```

线程池会在方便的时候尽早执行提交的任务。

下面总结了在使用连接线程池时做的工作：

1. 调用`Executors`类的静态方法`newCachedThreadPool`或者`newFixedThreadPool`。
2. 调用`submit`提交`Runnable`或`Callable`对象。
3. 保存好返回的`Future`对象，一遍得到结果或者取消任务。
4. 当不想再提交任何任务时，调用`shutdown`。

