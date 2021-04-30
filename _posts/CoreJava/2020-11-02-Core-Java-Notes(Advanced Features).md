---
layout:     post
title:      "Core Java - Daily Notes - 16"
subtitle:   " \"JAVA Knowledge - Advanced Features\""
date:       2020.11.02 12:29:00
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

## 6. 任务和线程池

### 6.3. 控制任务组

`invokeAll`方法提交一个`Callable`对象集合中的所有对象，这个方法会阻塞，直到所有任务都完成，并返回表示所有任务答案的一个`Future`对象列表，这一方法适用于有多个任务，但只要其中一个任务完成即可完成的情况：

```java
List<Callable<T>> tasks = ...;
List<Future<T>> results = executor.invokeAll(tasks);
for (Future<T> result : results)
    processFurther(result.get());
```

如果希望按照计算出结果的顺序得到这些结果，可以利用`ExcutorCompletionService`来管理。将任务提交到这个完成服务，该服务会管理`Future`对象的一个阻塞队列，其中包含所提交任务的结果（一旦结果可用，就会放入队列）。因此，以下组织更为高效：

```java
var service = new ExecutorCompletionService<T>(executor);
for (Callable<T> task : tasks)
	service.submit(task);
for (int i = 0; i < tasks.size(); i ++)
	processFurther(service.take().get());
```

如果要搜索包含指定单词的第一个文件，我们使用`invokeAny`来并行化这个搜索，在这里要注意任务的建立。一旦有任务返回，`invokeAny`方法就会终止。所以不能让搜索任务返回一个布尔值指示成功或者失败。我们不希望任务失败就停止搜索，而是应该抛出一个`NoSuchElementException`异常。另外，当一个任务成功时，其他任务就要取消。因此，我们要监视中断状态：

```java
public static Callable<Path> searchForTask(String word, Path path)
    {
        return () ->
        {
            try (var in = new Scanner(path))
            {
                while (in.hasNext())
                {
                    if (in.next().equals(word)) return path;
                    if (Thread.currentThread().isInterrupted())
                    {
                        System.out.println("Search in" + path + " cancled.");
                        return null;
                    }
                }
                throw new NoSuchElementException();
            }
        }
    }
```

如果需要打印执行期间线程池的最大大小，这个信息无法由`ExecutorService`接口提供，出于这个原因，我们必须把线程池对象强制转换为`Thread-PoolExecutor`类。

完整的查询文件中关键字的代码如下：

```java
package DailyTest;

import java.io.*;
import java.nio.file.*;
import java.time.*;
import java.util.*;
import java.util.concurrent.*;
import java.util.stream.*;

public class DailyTest
{
    /**
     * Counts occurrences of a given word in a file.
     * @return the number of times the word occurs in the given word
     */
    public static long occurrences(String word, Path path)
    {
        try (var in = new Scanner(path))
        {
            int count = 0;
            while (in.hasNext())
                if (in.next().equals(word)) count++;
            return count;
        }
        catch (IOException ex)
        {
            return 0;
        }
    }

    /**
     * Returns all descendants of a given directory--see Chapters 1 and 2 of Volume II.
     * @param rootDir the root directory
     * @return a set of all descendants of the root directory
     */
    public static Set<Path> descendants(Path rootDir) throws IOException
    {
        try (Stream<Path> entries = Files.walk(rootDir))
        {
            return entries.filter(Files::isRegularFile)
                    .collect(Collectors.toSet());
        }
    }

    /**
     * Yields a task that searches for a word in a file.
     * @param word the word to search
     * @param path the file in which to search
     * @return the search task that yields the path upon success
     */
    public static Callable<Path> searchForTask(String word, Path path)
    {
        return () -> {
            try (var in = new Scanner(path))
            {
                while (in.hasNext())
                {
                    if (in.next().equals(word)) return path;
                    if (Thread.currentThread().isInterrupted())
                    {
                        System.out.println("Search in " + path + " canceled.");
                        return null;
                    }
                }
                throw new NoSuchElementException();
            }
        };
    }

    public static void main(String[] args)
            throws InterruptedException, ExecutionException, IOException
    {
        try (var in = new Scanner(System.in))
        {
            System.out.print("Enter base directory (e.g. /opt/jdk-9-src): ");
            String start = in.nextLine();
            System.out.print("Enter keyword (e.g. volatile): ");
            String word = in.nextLine();

            Set<Path> files = descendants(Path.of(start));
            var tasks = new ArrayList<Callable<Long>>();
            for (Path file : files)
            {
                Callable<Long> task = () -> occurrences(word, file);
                tasks.add(task);
            }
            ExecutorService executor = Executors.newCachedThreadPool();
            // use a single thread executor instead to see if multiple threads
            // speed up the search
            // ExecutorService executor = Executors.newSingleThreadExecutor();

            Instant startTime = Instant.now();
            List<Future<Long>> results = executor.invokeAll(tasks);
            long total = 0;
            for (Future<Long> result : results)
                total += result.get();
            Instant endTime = Instant.now();
            System.out.println("Occurrences of " + word + ": " + total);
            System.out.println("Time elapsed: "
                    + Duration.between(startTime, endTime).toMillis() + " ms");

            var searchTasks = new ArrayList<Callable<Path>>();
            for (Path file : files)
                searchTasks.add(searchForTask(word, file));
            Path found = executor.invokeAny(searchTasks);
            System.out.println(word + " occurs in: " + found);

            if (executor instanceof ThreadPoolExecutor) // the single thread executor isn't
                System.out.println("Largest pool size: "
                        + ((ThreadPoolExecutor) executor).getLargestPoolSize());
            executor.shutdown();
        }
    }
}

```

