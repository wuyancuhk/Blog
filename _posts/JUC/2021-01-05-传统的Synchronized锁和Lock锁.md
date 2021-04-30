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

1. `synchronized`是内置的Java关键字，`Lock`是一个Java类；
2. 前者无法判断获取的锁的状态，`Lock`是可以的；
3. 前者会自动释放锁，后者必须要手动释放锁，不然会出现死锁问题；
4. 前者会导致另一个线程持续等待，后者可以通过`tryLock()`来尝试获取锁；
5. 前者是可重入锁，不可中断，是公平锁；后者也是可重入锁，可以判断锁，以及设置是否公平；
6. 前者适合锁少量的代码同步问题，后者适合大量的同步代码。

## 锁是什么，如何判断加锁的对象是什么

1. 生产者和消费者问题（等待->业务->通知）：

   - `synchronized`版本：

     ```java
     package org.lov3camille.trend;
     
     public class synchronizedTest {
     
         public static void main(String[] args) {
             Data data = new Data();
     
             new Thread(() -> {
                 for (int i = 0; i < 10; i ++) {
                     try {
                         data.increment();
                     } catch (InterruptedException e) {
                         e.printStackTrace();
                     }
                 }
             }, "A").start();
     
             new Thread(() -> {
                 for (int i = 0; i < 10; i ++) {
                     try {
                         data.decrement();
                     } catch (InterruptedException e) {
                         e.printStackTrace();
                     }
                 }
             }, "B").start();
     
             new Thread(() -> {
                 for (int i = 0; i < 10; i ++) {
                     try {
                         data.increment();
                     } catch (InterruptedException e) {
                         e.printStackTrace();
                     }
                 }
             }, "C").start();
         }
     
     }
     
     class Data {
         private int num = 0;
     
         public synchronized void increment() throws InterruptedException {
             if (num != 0) {
                 this.wait();
             }
             num ++;
     
             System.out.println(Thread.currentThread().getName() + "=>" + num);
     
             this.notifyAll();
         }
     
         public synchronized void decrement() throws InterruptedException {
             if (num == 0) {
                 this.wait();
             }
             num --;
     
             System.out.println(Thread.currentThread().getName() + "=>" + num);
     
             this.notifyAll();
         }
     }
     ```

     - 但上述的代码可能会导致“虚假唤醒"问题，需要将`if`判断改为`while`判断：

       ![image-20210107163339951](https://i.loli.net/2021/01/07/c7gOT86irfaxdjG.png)

       因为`if`只会执行一次，执行完会接着向下执行`if()`外边的 而`while`不会，直到条件满足才会向下执行`while()`外边的。
   
   - `Lock`方法：
   
     - ```java
       //官方示例
       class BoundedBuffer {
          final Lock lock = new ReentrantLock();
          final Condition notFull  = lock.newCondition(); 
          final Condition notEmpty = lock.newCondition(); 
       
          final Object[] items = new Object[100];
          int putptr, takeptr, count;
       
          public void put(Object x) throws InterruptedException {
            lock.lock();
            try {
              while (count == items.length)
                notFull.await();
              items[putptr] = x;
              if (++putptr == items.length) putptr = 0;
              ++count;
              notEmpty.signal();
            } finally {
              lock.unlock();
            }
          }
       
          public Object take() throws InterruptedException {
            lock.lock();
            try {
              while (count == 0)
                notEmpty.await();
              Object x = items[takeptr];
              if (++takeptr == items.length) takeptr = 0;
              --count;
              notFull.signal();
              return x;
            } finally {
              lock.unlock();
            }
          }
        }
       ```
   
     - 顺序打印“ABC”:
   
       ```java
       package org.lov3camille.trend;
       
       import java.util.concurrent.locks.Condition;
       import java.util.concurrent.locks.Lock;
       import java.util.concurrent.locks.ReentrantLock;
       
       public class ConditionTest {
           public static void main(String[] args) {
               Data2 data2 = new Data2();
       
               new Thread(()->{
                   for (int i = 0; i < 10; i ++) {
                       data2.printA();
                   }
               }, "A").start();
               new Thread(()->{
                   for (int i = 0; i < 10; i ++) {
                       data2.printB();
                   }
               }, "B").start();
               new Thread(()->{
                   for (int i = 0; i < 10; i ++) {
                       data2.printC();
                   }
               }, "C").start();
       
           }
       
       }
       
       class Data2 {
       
           private final Lock lock = new ReentrantLock();
           private final Condition condition1 = lock.newCondition();
           private final Condition condition2 = lock.newCondition();
           private final Condition condition3 = lock.newCondition();
           private int num = 1;
       
           public void printA() {
               lock.lock();
               try {
                   while (num != 1) {
                       condition1.await();
                   }
                   System.out.println(Thread.currentThread().getName() + "->A");
                   num = 2;
                   condition2.signal();
               } catch (InterruptedException e) {
                   e.printStackTrace();
               } finally {
                   lock.unlock();
               }
           }
       
           public void printB() {
               lock.lock();
               try {
                   while (num != 2) {
                       condition2.await();
                   }
                   System.out.println(Thread.currentThread().getName() + "->B");
                   num = 3;
                   condition3.signal();
               } catch (InterruptedException e) {
                   e.printStackTrace();
               } finally {
                   lock.unlock();
               }
           }
       
           public void printC() {
               lock.lock();
               try {
                   while (num != 3) {
                       condition3.await();
                   }
                   System.out.println(Thread.currentThread().getName() + "->C");
                   num = 1;
                   condition1.signal();
               } catch (InterruptedException e) {
                   e.printStackTrace();
               } finally {
                   lock.unlock();
               }
           }
       
       }
       
       ```
   
       





































