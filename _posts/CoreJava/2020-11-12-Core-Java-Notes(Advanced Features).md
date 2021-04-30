---
layout:     post
title:      "Core Java - Daily Notes - 19"
subtitle:   " \"JAVA Knowledge - Advanced Features\""
date:       2020.11.12 12:29:00
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

## 8. 进程

到目前为止，已经了解了如何在同一个程序的不同线程中执行`Java`代码。有时你还需要执行另一个程序。为此，可以使用`ProcessBuilder`和`Process`类。`Process`类在一个单独的操作系统进程中执行一个命令，允许我们与标准输入、输出和错误流交互。`ProcessBuilder`类则允许我们配置`Process`对象。

### 8.1. 建立一个进程

首先指定你想要执行的命令。可以提供一个`List<String>`，或者直接提供命令符字符串：

```java
var builder = new ProcessBuilder("gcc", "myapp.c");
```

不过要注意每一个字符串必须是可执行的，而不是`shell`的内置命令。

每个进程都有一个工作目录，用来解析相对目录名。默认情况下，进程的工作目录与虚拟机相同，通常都是启动`java`程序的那个目录。可以用`directory`的方法改变工作目录：

```java
builder = builder.directory(path.toFile());
```

接下来要指定如何处理进程的标准输入、输出和错误流，默认情况下可以通过如下方式访问：

```java
OutputStream processIn = p.getOutputStream();
InputStream processOut = p.getInputStream();
InputStream processErr = p.getErrorStream();
```

当然，也可以提供`File`对象，将进程流重定向到文件。

如果希望利用管道将一个进程的输出作为另一个进程的输入，`Java 9`提供了一个`startPipeline`方法，可以传入一个进程构建器列表，并从最后一个进程读取结果，示例如下，这里会枚举一个目录树中的各个扩展：

```java
List<Process> processes = ProcessBuilder.startPipeline(List.of(new ProcessBuilder("find", "/opt/jdf-9"), 
						  new ProcessBuilder("grep", "-o", "\\.[^./]*$"),
						  new ProcessBuilder("sort"),
						  new ProcessBuilder("uniq")
						  ));
```

### 8.2. 运行一个进程

配置了构建器后，要调用它的`start`方法来启动进程，例如：

```java
public static void main(String[] args) throws IOException {
        Process process = new ProcessBuilder("/bin/ls", "-l")
                .directory(Path.of("/tmp").toFile())
                .start();
        try (var in = new Scanner(process.getInputStream()))
        {
            while(in.hasNextLine())
                System.out.println(in.nextLine());
        }
    }
```

最后会在进程完成时收到一个异步通知。调用`process.onExit()`会得到一个`CompletableFuture<Process>`，可以用来调度任何动作。

### 8.3. 进程句柄

要获得程序启动的一个进程的更多信息，或者想更多地了解你的计算机上正在运行地任何其他进程，可以使用`ProcessHandle`接口。可以用4种方式得到一个`ProcessHandle`:

1. 给定一个`Process`对象`p`, `p.toHandle()`会生成它的`ProcessHandle`.
2. 给定一个`long`类型地操作系统进程ID，`ProcessHandle.of(id)`可以生成这个进程地句柄。
3. `Process.current()`是运行这个`Java`虚拟机的进程的句柄。
4. `ProcessHandle.allProcess()`可以生成对当前进程可见的所有操作系统进程的`Stream<ProcessHandle>`

注意，对句柄的一些返回信息的方法返回的实例只是当时的快照，在你看到它时可能就已经终止了。