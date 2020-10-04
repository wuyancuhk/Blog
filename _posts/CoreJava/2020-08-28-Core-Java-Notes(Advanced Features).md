---

layout:     post
title:      "Core Java - Daily Notes - 02"
subtitle:   " \"JAVA Knowledge - Advanced Features\""
date:       2020.08.28 10:00:00
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



### 1.3. 如何抛出异常

举个例子，如果一个承诺1024个字符的文档读取到733个字符就结束了，并且你认为这是不正常的，这时候就需要抛出一个异常。

具体语句如下：

```java
String readData(Scanner in) throws EOFException //指示输入过程中意外遇到了EOF（Java API 文档解释）
{
    ...
        while (...)
        {
            if (!in.hasNext()) // EOF encountered
            {
                if (n < len)
                    throw new EOFException();
            }
            ...
        }
        return s
}
```

`EOFException`类还有一个带一个字符串参数的构造器，可以用来更细致地描述异常：

```java
String gripe = "Content - length: " + len + ", Received: " + n;
throw new EOFException(gripe);
```

### 1.4. 创建异常类

自定义的类用来表达标准异常类无法描述的问题，其需要包含两个构造器，一个是默认的，一个是带有详细信息的。

例如：

```java
class FileForamtException extends IOException
{
    public FileFormatException(){}
    public FileFormatException(String gripe)
    {
        super(gripe)
    }
}
```

## 2. 捕获异常 

### 2.1. 捕获单个异常

如果发生了异常但是没有捕获，程序就会终止，并在控制台上打印一个消息，其中包括这个异常的类型和一个堆栈轨迹。

要想捕获一个异常，需要设置`try`/`catch`语句块，如下所示：

```java
try
{
    ... // main code body
}
catch (ExceptionType e)
{
    ... //handler for this type
}
```

如果`try`语句快中的任何代码抛出了`catch`字句中指定的一个异常类，那么：

1. 程序将跳过`try`语句块的其余代码；
2. 程序将执行`catch`字句中的处理器代码。

如果`try`语句块中的代码没有抛出任何异常，那么程序将跳过catch字句。

如果方法中的任何代码抛出了`catch`字句中没有声明的一个异常类型，那么这个方法就会立刻退出。

下面给出一个很典型的读取数据的代码：

```java
public void read(String filename)
{
    try
    {
        var in = new FileInputStream(filename);
        int b;
        while ((b = in.read()) != -1)
        {
            ... // process input
        }
    }
    catch (IOException exception)
    {
        exception.printStackTrace();
    }
}
```

另一种选择就是直接声明可能会抛出异常：

```java
public void read(String filename) throws IOException
{
    var in = new FileInputStream(filename);
    int b;
    while ((b = in.read()) != -1)
    {
        ... // process input
    }
}
```

对于这两种方法，需要自己决定是由自己处理还是抛出后交给处理器处理。

**当然，如果超类的方法没有抛出异常，子类的方法就必须捕获每一个检查型异常。**

### 2.2. 捕获多个异常

在一个`try`语句块中可以捕获多个异常类型，并对不同类型的异常做出不同的处理，要为每一个异常类型使用一个单独的`catch`字句。

异常对象可能包含有关异常性质的信息，要想获得关于这个对象的更多信息，可以尝试使用：

`e.getMessage()`

或者用`e.getClass().getName()`获取更详细的信息。

另外，当且仅当捕获的异常类型彼此之间不存在子类关系时，可以在同一个`catch`字句中捕获多个异常类型。

捕获多个异常对应生成的字节码对应公共`catch`语句的一个代码块，因此会让代码更简洁高效。

### 2.3. 再次抛出异常与异常链

可以用这种方式指示子系统故障的异常类型，而非关注于如何处理它，例如：

```java
try
{
    ... // access the database
}
catch (SQLException e)
{
    throw new ServletException("database error: " + e.getMessage());
}
```

或者将原始异常设置为新异常的“原因”：

```java
try
{
    ... // access the database
}
catch (SQLException original)
{
    var e = new ServletException("database error");
    e.initCause(original);
    throw e;
}
```

从而可以利用`Throwable original = caughtException.getCause();`来获取原始异常，这样可以在抛出高层异常的同时不会丢失原始异常的细节。也可以用到当检查型异常不允许抛出的时候，将其包装成一个运行时异常。

### 2.4. finally子句

如果抛出异常后有些剩余的资源必须得到清理，就需要用到`finally`子句。

程序有可能在以下三种情况下执行`finally`子句：

1. 代码没有抛出异常，则执行顺序为`try` -> `finally`;

2. 代码抛出了一个异常，并在一个`catch`语句中捕获：

   2.1. 如果`catch`子句没有抛出异常，则程序会执行`finally`子句后的第一条语句；

   2.2. 如果`catch`子句抛出了一个异常，则异常将呗抛回到这个方法的调用者。

3. 代码抛出了一个异常，但是没有任何`catch`子句捕获这个异常，程序会跳过`try`语句中的剩余代码，然后执行`finally`子句中的语句，并将异常抛回给这个方法的调用者。

警告：`finally`子句的体要用于清理资源，不要把改变控制流的语句（`return`, `throw`, `break`, `continue`）放在`finally`子句中，以免遮蔽原先的返回值。

### 2.5. try-with-Resources 语句

`try`块正常退出时，或存在一个异常时，都会调用`in.close()`之类的方法，就好像用了`finally`块一样，同时还可以指定多个资源，例如：

```java
public class Main{
        public static void main(String[] args)
        {

                try (var in = new Scanner(new FileInputStream("/Java/src/DailyTest/test"), StandardCharsets.UTF_8);
                var out = new PrintWriter("out.txt", StandardCharsets.UTF_8))
                {
                        while (in.hasNext())
                                out.println(in.next().toUpperCase());
                }
                catch (Exception e)
                {
                        e.printStackTrace();
                }
        }
}
```

在Java 9中，也可以在`try`首部中提供之前声明的事实最终变量。

这种方法解决的问题是：如果`try`块抛出一个异常，而且`close`方法也抛出一个异常。

这样的话，`close`方法抛出的异常会“被抑制”，捕获的异常将由`addSuppressed`方法增加到原来的异常，感兴趣的话可以通过`getSuppressed`方法生成从`close`方法抛出并被抑制的异常**数组**。当然，这一语句本身也是有`catch`或者`finally`子句的，他们都会在资源关闭之后执行。