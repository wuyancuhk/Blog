---
layout:     post
title:      "JUC - 传统的Synchronized锁和Lock锁"
subtitle:   " \"JUC Daily Notes - 02\""
date:       2021.01.05 12:29:00
author:     "Wuy"
header-img: "img/post-bg-2020.jpg"
catalog: true
tags:
    - 学习笔记
    - 读书
    - JUC


---

> *"Keep Learning JUC"*

# 锁机制

## `Synchronized`锁

以下是该锁的示例：

```java
package org.lov3camille.trend;

import static org.junit.Assert.assertTrue;

import org.apache.hadoop.hdfs.web.JsonUtil;
import org.junit.Test;

import java.util.ArrayList;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class AppTest 
{
    public static void main(String[] args) {
        ticket ti = new ticket();
        new Thread(() -> {
            for (int i = 0; i < 60; i ++) {
                ti.sale();
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < 60; i ++) {
                ti.sale();
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < 60; i ++) {
                ti.sale();
            }
        }, "C").start();
    }
}
class ticket {
    private int num = 50;
    private int i = 0;

    Lock lock = new ReentrantLock();
    public synchronized void sale() {
        if (num > 0) {
            num--;
            i++;
            System.out.println(Thread.currentThread().getName() + i + "tickets sold" + num + " ;tickets left");
        }
    }
}

```



## `Lock`锁

1. `ReetrantLock`可重入锁：

   ![image-20210105221144085](https://i.loli.net/2021/01/05/BeXSxOq3hEVM8LP.png)

   其中，公平锁：十分公平，可以先来后到；

   **非公平锁：十分不公平，可以插队（Java默认）**

   随着这种增加的灵活性，也产生了额外的责任。 没有块结构化锁定会删除使用`synchronized`方法和语句发生的锁的自动释放。 在大多数情况下，应使用以下惯用语：

   ```java
   Lock l = ...; l.lock(); try { /* access the resource protected by this lock */} finally { l.unlock(); } 
   ```

   当在不同范围内发生锁定和解锁时，必须注意确保在锁定时执行的所有代码由`try-finally`或`try-catch`保护，以确保在必要时释放锁定。

   将上面的例子重写：

   ```java
   package org.lov3camille.trend;
   
   import static org.junit.Assert.assertTrue;
   
   import org.apache.hadoop.hdfs.web.JsonUtil;
   import org.junit.Test;
   
   import java.util.ArrayList;
   import java.util.concurrent.locks.Lock;
   import java.util.concurrent.locks.ReentrantLock;
   
   public class AppTest 
   {
       public static void main(String[] args) {
           ticket ti = new ticket();
           new Thread(() -> {
               for (int i = 0; i < 60; i ++) {
                   ti.sale();
               }
           }, "A").start();
   
           new Thread(() -> {
               for (int i = 0; i < 60; i ++) {
                   ti.sale();
               }
           }, "B").start();
   
           new Thread(() -> {
               for (int i = 0; i < 60; i ++) {
                   ti.sale();
               }
           }, "C").start();
       }
   }
   class ticket {
       private int num = 50;
       private int i = 0;
   
       Lock lock = new ReentrantLock();
   
       public void sale() {
           lock.lock();
   
           try {
               if (num > 0) {
                   num--;
                   i++;
                   System.out.println(Thread.currentThread().getName() + i + "tickets sold" + num + " ;tickets left");
               }
           } catch (Exception e) {
               e.printStackTrace();
           } finally {
               lock.unlock();
           }
       }
   }
   
   ```

   

2. `ReadWriteLock`读写锁：

   ![image-20210105223020496](https://i.loli.net/2021/01/05/dJq6ID43HWTVCv1.png)

## `synchronized`锁和`Lock`锁的区别









































