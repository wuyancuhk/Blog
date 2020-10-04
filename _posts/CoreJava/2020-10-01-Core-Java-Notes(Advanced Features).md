---
layout:     post
title:      "Core Java - Daily Notes - 13"
subtitle:   " \"JAVA Knowledge - Advanced Features\""
date:       2020.10.01 12:29:00
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

### 5.2. 高效的映射、集和队列

`java.util.concurrent`包提供了映射、有序集和队列的高效实现：`ConcrrentHashMap`, `ConcurrerntSkipListMap`, `ConcurrentSkipListSet`, `ConcurrentLinkedQueue`。

这些集合使用复杂的算法，通过允许并发地访问数据结构地不同部分尽可能减少竞争。

但这些类的`size`方法不一定在常量时间内完成，确定它们的大小通常需要遍历。

集合返回弱一致性的迭代器，这意味着不一定能反映出它们构造后的所有更改，但是，它们不会将一个值返回两次，也不会抛出`ConcurrentModificationException`异常。

并发散列映射可以高效地支持大量地阅读器和一定数量地书写器，默认情况下可以有至多16个同时运行地书写器线程。

散列映射会将所有相同散列码的所有条目放在同一个桶中，这会严重影响性能，在较新的`Java`版本中，并发散列映射实现了将桶组织为树，而不是列表，键类型实现`Comparable`，从而可以保证性能为`O(log(n))`。

### 5.3. 映射条目的原子更新

`ConcurrentHashMap`原来的版本只有为数不多的方法可以实现原子更新，这使得编程有些麻烦。

比如下面的让计数自增的代码就不是线程安全的：

```java
Long oldvalue = map.get(word);
Long newvalue = oldvalue == null ? : oldvalue + 1;
map.put(word, newvalue);
```

可能会有另一个线程在同时更新同一个计数。当然，为什么线程安全的数据结构也会允许这种操作呢，首先`get`和`put`方法并不会破坏数据结构（但会破坏普通的`HashMap`），`ConcurrentHashMap`绝对不会，但是由于操作序列不是原子的，所以结果不可预知。

如今，`JAVA API`提供了一些新的方法，可以更方便地完成原子更新。例如，可以如下更新一个整数计数器的映射：

```java
map.compute(word, (k, v) -> v == null ? 1 : v + 1);
```

同时注意，`ConcurrentHashMap`中不允许有`null`值，很多方法都使用`null`值来指示映射中某个给定的键不存在。

另外还有`ComputeIfAbsent()`方法，它只在没有原值的情况下计算新值，比如如下更新一个`LongAdder`计数器映射：

```
map.computeIfAbsent(word, k -> new LongAdder().increment());
```

下面的程序使用了一个并发散列映射来统计一个目录树的`Java`文件中的所有单词：

```java
package DailyTest;

import java.io.*;
import java.nio.file.*;
import java.util.*;
import java.util.concurrent.*;
import java.util.stream.*;

public class DailyTest
{
    public static ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();

    /**
     * Adds all words in the given file to the concurrent hash map.
     * @param file a file
     */
    public static void process(Path file)
    {
        try (var in = new Scanner(file))
        {
            while (in.hasNext())
            {
                String word = in.next();
                map.merge(word, 1L, Long::sum);
            }
        }
        catch (IOException e)
        {
            e.printStackTrace();
        }
    }

    /**
     * Returns all descendants of a given directory--see Chapters 1 and 2 of Volume II
     * @param rootDir the root directory
     * @return a set of all descendants of the root directory
     */
    public static Set<Path> descendants(Path rootDir) throws IOException
    {
        try (Stream<Path> entries = Files.walk(rootDir))
        {
            return entries.collect(Collectors.toSet());
        }
    }

    public static void main(String[] args)
            throws InterruptedException, ExecutionException, IOException
    {
        int processors = Runtime.getRuntime().availableProcessors();
        ExecutorService executor = Executors.newFixedThreadPool(processors);
        Path pathToRoot = Path.of(".");
        for (Path p : descendants(pathToRoot))
        {
            if (p.getFileName().toString().endsWith(".java"))
                executor.execute(() -> process(p));
        }
        executor.shutdown();
        executor.awaitTermination(10, TimeUnit.MINUTES);
        map.forEach((k, v) ->
        {
            if (v >= 10)
                System.out.println(k + " occurs " + v + " times");
        });
    }
}
```

### 5.4. 对并发散列映射的批操作

批操作会遍历映射，处理遍历过程中找到的元素，这里不会冻结当前快照，除非你恰好知道批操作运行时映射不会被修改，否则就要把结果看作是映射状态的一个近似。

有三种不同的操作：

- `search`:也就是搜索，为每个键或值应用一个函数，直到函数生成一个非空的结果，然后返回。
- `reduce`:也就是归约， 组合所有的键或值，要使用一个累加函数。
- `forEach`:为所有的键或值应用一个函数。

每个操作都有四个版本：

- *operationKeys*: 处理键；
- *operationValues*：处理值；
- *operation*：处理键和值；
- *operationEntries*：处理`Map.Entries`对象。

对于上述的各个操作，都需要指定一个参数化阈值。如果映射包含的元素多于这个阈值，就会并行地完成批操作，也就是说，如果希望只在一个线程中运行，就将阈值设为`Long.MAX_VALUE`，如果希望尽可能多的线程运行，就将阈值设为1。

比如，希望找到第一个出现次数大于1000的单词：

```java
String result = map.search(threshold, (k, v) -> v > 1000 ? k : null);
```

`result`会设置为第一个匹配的单词，或者所有输入都为`null`时则返回`null`。

`forEach`方法则有两种形式，第一种形式只对各个映射条目应用一个消费者函数，例如：

```java
map.forEach(threshold, (k, v) -> System.out.println(k + " -> " + v));
```

第二种形式还有一个额外的转换器作为参数，要先应用这个函数，其结果会传递到消费者：

```java
map.forEach(threshold, (k, v) -> k + " -> " + v, System.out::println);
```

