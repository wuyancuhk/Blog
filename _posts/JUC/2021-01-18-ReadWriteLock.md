---
layout:     post
title:      "JUC - ReadWriteLock"
subtitle:   " \"JUC Daily Notes - 06\""
date:       2021.01.18 12:29:00
author:     "Wuy"
header-img: "img/post-bg-2020.jpg"
catalog: true
tags:
    - 学习笔记
    - 读书
    - JUC
---

> *"Keep Learning JUC"*

# `ReadWriteLock`

读写锁的特性简单地讲就是可以被多线程同时读取，但是只能被单个线程写入。

![image-20210118163602516](https://i.loli.net/2021/01/18/ZNztY9CL8U6s1OJ.png)



示例如下：

```java
package org.example;

import org.apache.hadoop.classification.InterfaceAudience;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

// 独占锁（写锁）， 共享锁（读锁）
// 对于读写锁，
// 读-读 可以共存；
// 读-写 不可以共存；
// 写-写 不可以共存。
public class ReadWriteLockTest {
    public static void main(String[] args) {
        Mycache mycache = new Mycache();

        MycacheLock mycacheLock = new MycacheLock();
//        // read
//        for (int i = 0; i < 6; i++) {
//            final int temp = i;
//            new Thread(() -> {
//                mycache.put(temp + "", temp + "");
//            }).start();
//        }
//
//        // write
//        for (int i = 0; i < 6; i++) {
//            final int temp = i;
//            new Thread(() -> {
//                mycache.get(temp + "");
//            }).start();
//        }

        // lock read
        for (int i = 0; i < 6; i++) {
            final int temp = i;
            new Thread(() -> {
                mycacheLock.put(temp + "", temp + "");
            }).start();
        }

        // lock write
        for (int i = 0; i < 6; i++) {
            final int temp = i;
            new Thread(() -> {
                mycacheLock.get(temp + "");
            }).start();
        }

    }
}

// 自定义缓存
class Mycache {

    private volatile Map<String, Object> map = new HashMap<>();


    // 存，写；
    public void put(String key, Object value) {
        System.out.println(Thread.currentThread().getName() + "put " + key);
        map.put(key, value);
        System.out.println(Thread.currentThread().getName() + "put finish");
    }

    // 取，读；
    public void get(String key) {
        System.out.println(Thread.currentThread().getName() + "read" + key);
        Object o = map.get(key);
        System.out.println(Thread.currentThread().getName() + "read finish");
    }
}

// 加锁缓存
class MycacheLock {

    private volatile Map<String, Object> map = new HashMap<>();

    // 读写锁，更加细粒度的控制
    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    // 存，写入的时候，只希望同时只有一个线程写；
    public void put(String key, Object value) {
        readWriteLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "put " + key);
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "put finish");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.writeLock().unlock();
        }
    }

    // 取，读， 所有人都可以读；
    public void get(String key) {
        readWriteLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "read" + key);
            Object o = map.get(key);
            System.out.println(Thread.currentThread().getName() + "read finish");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.readLock().unlock();
        }
    }
}

```

















































