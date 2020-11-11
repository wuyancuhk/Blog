---
layout:     post
title:      "Core Java - Daily Notes - 18"
subtitle:   " \"JAVA Knowledge - Advanced Features\""
date:       2020.11.11 12:29:00
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

## 7. 异步计算

### 7.1. 可完成`future`

当有一个`Future`对象时，需要调用`get`来获得值，这个方法会阻塞，直到值可用。`CompletableFuture`类实现了`Future`接口，它提供了获得结果的另一种机制。通过这种方式，无需阻塞就可以在结果可用时对结果进行回调。

要想异步运行任务并得到`CompletableFuture`，不要把它直接提交给执行器服务，而应当调用静态方法`CompletableFuture.supplyAsync`。

这个对象有两种完成方式：得到一个结果或者一个未捕获的异常，可以使用`whenComplete`方法来得到。

### 7.2. 组合可完成`future`

如果下一个动作也是异步的，那么在它的下一个动作就会在一个不同的回调中，并且程序逻辑会分散到不同的回调中，再加上错误处理的话，情况会更糟糕。

假设有这样一个方法：

```java
public void CompletableFuture<String> readPage(URL url)
```

Web页面可用时，这会生成这个页面的文本。如果方法：

```java
public List<URL> getImageURLs(String page)
```

可以生成一个HTML页面中图像的URL，可以调度当页面可用时要调用的这个方法：

```java
CompletableFuture<String> contents = readPage(url);
CompletableFuture<List<URL>> imageURLs = contents.thenApply(this :: getLinks);
```

该方法不会阻塞，它会返回另一个`future`，第一个`future`完成时，其结果会提供给`getImageURLs`方法，这个方法的返回值就是最终的结果。

最后的静态`allof`和`anyof`方法取一组可完成`future`（数目可变），并生成一个`CompletableFuture<void>`，它会在所有这些`future`都完成时或者其中任意一个`future`完成时结束。`allof`方法不会生成任何结果，`anyof`方法不会终止其余的任务。

下面的程序清单表示读取一个Web页面、扫描页面得到其中的图像并且保存本地。

```java
package completableFutures;

import java.awt.image.*;
import java.io.*;
import java.net.*;
import java.nio.charset.*;
import java.util.*;
import java.util.concurrent.*;
import java.util.regex.*;

import javax.imageio.*;

public class CompletableFutureDemo
{
   private static final Pattern IMG_PATTERN = Pattern.compile(
      "[<]\\s*[iI][mM][gG]\\s*[^>]*[sS][rR][cC]\\s*[=]\\s*['\"]([^'\"]*)['\"][^>]*[>]");
   private ExecutorService executor = Executors.newCachedThreadPool();
   private URL urlToProcess;

   public CompletableFuture<String> readPage(URL url)
   {
      return CompletableFuture.supplyAsync(() -> 
         {
            try
            {
               var contents = new String(url.openStream().readAllBytes(),
                  StandardCharsets.UTF_8);
               System.out.println("Read page from " + url);
               return contents;
            }
            catch (IOException e)
            {
               throw new UncheckedIOException(e);
            }
         }, executor);
   }

   public List<URL> getImageURLs(String webpage) // not time-consuming
   { 
      try
      {
         var result = new ArrayList<URL>();
         Matcher matcher = IMG_PATTERN.matcher(webpage);
         while (matcher.find())
         {
            var url = new URL(urlToProcess, matcher.group(1));
            result.add(url);
         }
         System.out.println("Found URLs: " + result);
         return result;
      }
      catch (IOException e)
      {
         throw new UncheckedIOException(e);
      }
   }

   public CompletableFuture<List<BufferedImage>> getImages(List<URL> urls)
   {
      return CompletableFuture.supplyAsync(() -> 
         {
            try
            {
               var result = new ArrayList<BufferedImage>();
               for (URL url : urls)
               {
                  result.add(ImageIO.read(url));
                  System.out.println("Loaded " + url);
               }
               return result;
            }
            catch (IOException e)
            {
               throw new UncheckedIOException(e);
            }
         }, executor);
   }

   public void saveImages(List<BufferedImage> images)
   {
      System.out.println("Saving " + images.size() + " images");
      try
      {
         for (int i = 0; i < images.size(); i++)
         {
            String filename = "/tmp/image" + (i + 1) + ".png";
            ImageIO.write(images.get(i), "PNG", new File(filename));
         }
      }
      catch (IOException e)
      {
         throw new UncheckedIOException(e);
      }
      executor.shutdown();
   }

   public void run(URL url)
         throws IOException, InterruptedException
   {
      urlToProcess = url;
      CompletableFuture.completedFuture(url)
         .thenComposeAsync(this::readPage, executor)
         .thenApply(this::getImageURLs) 
         .thenCompose(this::getImages)
         .thenAccept(this::saveImages); 
         
      /* 
      // or use the experimental HTTP client:
       
      HttpClient client = HttpClient.newBuilder().executor(executor).build();
      HttpRequest request = HttpRequest.newBuilder(urlToProcess.toURI()).GET()
         .build();
      client.sendAsync(request, BodyProcessor.asString())
         .thenApply(HttpResponse::body).thenApply(this::getImageURLs)
         .thenCompose(this::getImages).thenAccept(this::saveImages);
      */
   }

   public static void main(String[] args) 
         throws IOException, InterruptedException
   {
      new CompletableFutureDemo().run(new URL("http://horstmann.com/index.html"));
   }
}
```

