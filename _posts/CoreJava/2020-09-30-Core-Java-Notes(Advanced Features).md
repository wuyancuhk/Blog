---
layout:     post
title:      "Core Java - Daily Notes - 12"
subtitle:   " \"JAVA Knowledge - Advanced Features\""
date:       2020.09.30 12:29:00
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

### 4.13. 为什么废弃`stop`和`suspend`方法

这两个方法有个共同点，就是都试图控制一个给定线程的行为，而没有线程的互操作。

首先来看看`stop`方法，这个方法会终止所有未结束的方法，包括`run`方法，当线程被终止，它会立即释放被它锁定的所有对象的锁，这会导致对象处于不一致的状态。也就是说，它会破坏对象，这是不安全的。

另外，被停止的对象会抛出一个`ThreadDeath`异常，从而推出它调用的所有同步方法，因此，这个线程会释放它持有的内部对象锁，所以说，不能说`stop`方法会导致对象被一个已停止的线程永久锁定。

接下来看看`suspend`方法。如果用该方法挂起一个持有锁的线程，那么，在线程恢复运行前这个锁是不可用的。如果调用该方法的线程试图获得同一个锁，那么程序就会死锁：被挂起的线程等着被恢复，而将其挂起的线程等待获得锁。

解决的方法是引入一个`suspendRequested`变量，并在`run`方法的某个安全的地方测试，安全的地方是指该线程没有锁定其他线程需要的对象，当该线程发现这一变量已经被设置了，就会继续等待，直到再次可用。

## 5. 线程安全的集合

如果多个线程要并发地修改一个数据结构，那么很容易破坏这个数据结构，可以提供锁来保护共享的数据结构，但是选择线程安全的实现可能更为容易。

### 5.1. 阻塞队列

当然，线程安全的队列类的实现者必须考虑锁和条件，但那是他们的问题，不是我的问题。当试图向队列添加元素而队列已满，或者想从队列移出元素而队列为空时，**阻塞队列**将导致线程阻塞。

在多线程程序中，队列可能会在任何时候变空或者变满，因此，应当使用`offer`、`poll`、`peek`方法作为替代，如果不能完成任务，它们只会返回一个错误提示而不是抛出一个异常。（向队列中插入`null`值是非法的）

常见的阻塞队列都在`java.util.concurrent`包里，比如`LinkedBlockingQueue`, `ArrayBlockingQueue`, `PriorityBlockingQueue`, `DelayQueue`等等。

下面的程序展示了如何使用阻塞队列来控制一组线程，该程序实现了在一个目录及其所有子目录下搜索所有文件，打印出包含指定关键字的行：

```java
package DailyTest;

import java.io.*;
import java.nio.charset.*;
import java.nio.file.*;
import java.util.*;
import java.util.concurrent.*;
import java.util.stream.*;

public class Test
{
    private static final int FILE_QUEUE_SIZE = 10;
    private static final int SEARCH_THREADS = 100;
    private static final Path DUMMY = Path.of("");
    private static BlockingQueue<Path> queue = new ArrayBlockingQueue<>(FILE_QUEUE_SIZE);

    public static void main(String[] args)
    {
        try (var in = new Scanner(System.in))
        {
            System.out.print("Enter base directory (e.g. /opt/jdk-9-src): ");
            String directory = in.nextLine();
            System.out.print("Enter keyword (e.g. volatile): ");
            String keyword = in.nextLine();

            Runnable enumerator = () -> {
                try
                {
                    enumerate(Path.of(directory));
                    queue.put(DUMMY);
                }
                catch (IOException e)
                {
                    e.printStackTrace();
                }
                catch (InterruptedException e)
                {
                }
            };

            new Thread(enumerator).start();
            for (int i = 1; i <= SEARCH_THREADS; i++) {
                Runnable searcher = () -> {
                    try
                    {
                        var done = false;
                        while (!done)
                        {
                            Path file = queue.take();
                            if (file == DUMMY)
                            {
                                queue.put(file);
                                done = true;
                            }
                            else search(file, keyword);
                        }
                    }
                    catch (IOException e)
                    {
                        e.printStackTrace();
                    }
                    catch (InterruptedException e)
                    {
                    }
                };
                new Thread(searcher).start();
            }
        }
    }

    /**
     * Recursively enumerates all files in a given directory and its subdirectories.
     * See Chapters 1 and 2 of Volume II for the stream and file operations.
     * @param directory the directory in which to start
     */
    public static void enumerate(Path directory) throws IOException, InterruptedException
    {
        try (Stream<Path> children = Files.list(directory))
        {
            for (Path child : children.collect(Collectors.toList()))
            {
                if (Files.isDirectory(child))
                    enumerate(child);
                else
                    queue.put(child);
            }
        }
    }

    /**
     * Searches a file for a given keyword and prints all matching lines.
     * @param file the file to search
     * @param keyword the keyword to search for
     */
    public static void search(Path file, String keyword) throws IOException
    {
        try (var in = new Scanner(file, StandardCharsets.UTF_8))
        {
            int lineNumber = 0;
            while (in.hasNextLine())
            {
                lineNumber++;
                String line = in.nextLine();
                if (line.contains(keyword))
                    System.out.printf("%s:%d:%s%n", file, lineNumber, line);
            }
        }
    }
}

```

生产者线程枚举所有子目录下的所有文件并把他们放到一个阻塞队列中，这个操作很快。