---

layout:     post
title:      "Core Java - Daily Notes - 03"
subtitle:   " \"JAVA Knowledge - Advanced Features\""
date:       2020.08.31 17:00:00
author:     "Wuy"
header-img: "img/post-bg-2020.jpg"
catalog: true
tags:
    - 学习笔记
    - 读书
    - JAVA

---

> *"Keep Learning JAVA"*

# 异常、断言和日志



### 2.6. 分析堆栈轨迹元素

堆栈轨迹是程序执行过程中某个特定点上所有挂起的方法调用的一个列表。

可以调用`Throwable`类的`printStackTrace`方法访问堆栈轨迹的文本描述信息。

```java
var t = new Throwable();
var out = new StringWriter();
t.printStackTrace(new PrinWriter(out));
String description = out.toString();
```

另一种更灵活的方法是使用`StackWalker`类， 它会生成一个`StackWalker.StackFrame`实例流，其中每个实例分别描述一个栈帧。可以利用下面的迭代来处理这些栈帧：

```java
StackWalker walker = StackWalker.getInstance();
walker.forEach(frame -> 'analyze' frame)
```

简单的调用例子如下：

```java
package stackTrace;

import java.util.*;

public class StackTraceTest
{
   public static int factorial(int n)
   {
      System.out.println("factorial(" + n + "):");
      var walker = StackWalker.getInstance();
      walker.forEach(System.out::println);      
      int r;
      if (n <= 1) r = 1;
      else r = n * factorial(n - 1);
      System.out.println("return " + r);
      return r;
   }

   public static void main(String[] args)
   {
      try (var in = new Scanner(System.in))
      {
         System.out.print("Enter n: ");
         int n = in.nextInt();
         factorial(n);
      }
   }
}

```

## 3. 使用异常的技巧

存在的就是有意义的，下面给出一些使用异常的技巧：

1. 异常处理不能代替简单的测试

   测试代码：

   ```java
   if (!s.empty()) s.pop(); // 646ms 
   ```

   捕获异常代码：

   ```java
   try
   {
       s.pop();
   }
   catch (EmptyStackException e)
   {
       ...
   } // 21739ms
   ```

   

2. 不要过分地细化异常

   不要将每一条语句都分装在一个独立地`try`语句块中。这种编程方式会导致代码量地急剧膨胀。注意一个原则：要将正常处理与错误处理分开。

3. 充分利用异常层次结构

   不要只抛出`RuntimeException`异常，应该寻找一个适合的子类或创建自己的异常类；

   不要只捕获`Throwable`异常，这样会使代码更难读，更难维护。

4. 不要压制异常

5. 在检测错误时，“苛刻”要比放任更好

   比如说，当栈为空时，最好是抛出一个`EmptyStackException`异常，而不是抛出一个`null`，这样以后还是会抛出`NullPointerExcepyion`异常，得不偿失。

6. 不要羞于传递异常（早抛出，晚捕获）
