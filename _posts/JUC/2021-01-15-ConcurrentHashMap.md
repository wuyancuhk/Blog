---
layout:     post
title:      "JUC - ConcurrentHashMap"
subtitle:   " \"JUC Daily Notes - 04\""
date:       2021.01.15 12:29:00
author:     "Wuy"
header-img: "img/post-bg-2020.jpg"
catalog: true
tags:
    - 学习笔记
    - 读书
    - JUC

---

> *"Keep Learning JUC"*

1. 关于`HashMap`加载因子及转红黑树探究:

   参考这篇文章：https://blog.csdn.net/refuse_debug/article/details/104623908

2. 原始`HashMap`线程不安全的测试：

   ```java
   package org.lov3camille.trend;
   
   
   import java.util.*;
   import java.util.concurrent.CopyOnWriteArraySet;
   
   public class MapTest {
       public static void main(String[] args) {
           Map<String, String> map = new HashMap<>();
           for (int i = 1; i < 10; i++) {
               new Thread(() -> {
                   map.put(Thread.currentThread().getName(), UUID.randomUUID().toString().substring(0, 5));
                   System.out.println(map);
               }, String.valueOf(i)).start();
           }
       }
   }
   
   ```

   修改方式也和之前的`COW`类似，但注意，第二种方法变为了使用`ConcurrentHashMap`。

   这里我们通过官方文档探究`ConcurrentHashMap`实现并发安全的原理。

