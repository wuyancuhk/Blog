---
layout:     post
title:      "Core Java - Daily Notes - 14"
subtitle:   " \"JAVA Knowledge - Advanced Features\""
date:       2020.10.05 12:29:00
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

### 5.5. 并发集视图

假设你想要的是一个很大的线程安全的集而不是线程，并没有`ConcurrentHashSet`类，而你肯定也不想自己创建这样一个类。

静态`newKeySet`方法会生成一个`Set<K>`，这实际上是`ConcurrentHashMap<K, Boolean>`的一个包装器（所有的映射值都为`True`）。

用法如下：

```java
Set<String> words = ConcurrentHashMap.<String>newKeySet();
```

当然，如果原来有一个映射，那么`keySet`方法可以生成这个映射的键集，这个集是可以更改的，但是删除可以（同时也会删除映射中的键值对），增加元素是不行的，因为没有相应的值可以增加。当然，还有一个带默认值的方法，如果增加的元素不存在，那么就新增一个默认值：

```java
Set<String> words = map.KeySet(1L);
words.add("Java");//这里“Java”不存在，所以会有一个值1.
```

### 5.6. 写数组的拷贝

`CopyOnwriteArrayList`和`CopyOnWriteArraySet`是线程安全的集合其中所有的更改器会建立底层数组的一个副本，如果迭代访问集合的线程数超过更改集合的线程数，这样的安排是很有用的。当构造一个迭代器时，它包含当前数组的一个引用，如果这个数组后来被更改了，迭代器仍然引用旧数组，但是，集合的数组已经被替换。从而，原来的迭代器可以访问一致的但也有可能是过时的数组，而不存在任何同步的开销。

### 5.7. 并行数组算法

`Arrays`类提供了大量并行化操作，静态`Array.parallelSort`方法可以对一个基本类型值或者对象的数组排序，例如：

```java
String contents = new String(Files.readAllBytes(Path.of("alice.txt")), StandardCharsets.UTF_8);
String[] words = contents.split("[\\P{L}]+");
Arrays.parallelSort(words);
```

对对象排序时，可以提供一个`Comparator`：

```java
Arrays.parallelSort(words, Comparator.comparing(String::length));
```

`parallelSetAll`方法会用由一个函数计算得到的值填充一个数组，这个函数接收元素索引，然后计算相应位置上的值：

```java
Arrays.parallelSetAll(values, i -> i % 10);
```

显然，并行化对这个操作很有好处，这个操作对于所有的基本类型数组和对象数组都有相应的版本。

最后还有个`parallelPrefix()`方法，它会用一个给定操作的相应前缀的累加结果替换各个数组元素：

```java
public static void main(String[] args) throws IOException {

        int[] array = new int[11];
        for (int i = 0; i <= 10; i ++)
        {
            array[i] = i + 1;
        }
        Arrays.parallelPrefix(array, (x, y) -> x * y);
        System.out.println(Arrays.toString(array));
    }
```

输出如下：

```
[1, 2, 6, 24, 120, 720, 5040, 40320, 362880, 3628800, 39916800]
```

这同样可以并行完成，并且时间复杂度为`O(log(n))`。

### 5.8. 较早的线程安全集合

集合类库中提供了一种不同的机制，任何集合类都可以通过使用同步包装器变成线程安全的：

```java
 List<String> synchArrayList = Collections.synchronizedList(new ArrayList<String>());
 Map<String, Integer> syncHashMap =Collections.synchronizedMap(new HashMap<String, Integer>());
```

结果集合的方法使用锁加以保护，可以提供线程安全的访问。

但是如果希望迭代地访问一个集合，同时另一个线程仍有机会更改这个集合，那么仍然需要“客户端”锁定,不然迭代器会失效，并抛出`ConcurrentModificationException`异常：

```java
synchronized(syncHashMap)
{
	Iterator<K> iter - syncHashMap.keySet().iterator();
	while (iter.hasNext())
	...
}
```

当然，多个线程访问不同的桶时，最好用`ConcurrentHashMap`，除非数组列表经常更改，那么这时用同步的`ArrayList`要好点。