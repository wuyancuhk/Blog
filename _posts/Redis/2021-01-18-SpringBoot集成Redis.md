---
layout:     post
title:      "Redis - SpringBoot集成Redis"
subtitle:   " \"Redis Knowledge - 04\""
date:       2021.01.18 12:29:00
author:     "Wuy"
header-img: "img/post-bg-2020.jpg"
catalog: true
tags:
    - 学习笔记
    - 读书
    - Redis
---

> *"Keep Learning Redis"*

# SpringBoot整合Redis

SpringData也是和SpringBoot齐名的项目，在SpringBoot2.X之后，原来的Jedis被替换为了现在的lettuce.

原因包括：

- Jedis： 采用的直连，多个线程操作的话，是不安全的，如果想要避免不安全，使用Jedis pool连接池，类似于BIO模式；
- lettuce：底层采用Netty， 实例可以在多个线程下共享，不存在线程不安全的情况，可以减少线程数据量，类似于NIO模式。

SpringBoot所有的配置类都有一个自动配置类（`RedisAutoConfiguration`），自动配置类都会绑定一个`properties`配置文件（`RedisProperties`）；

下面看下`RedisAutoConfiguration.class`, 这是默认的设置，但一般需要加一个序列化的操作，另外还需要强制类型转换，所以可以自己写一个来替换原生的类:

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package org.springframework.boot.autoconfigure.data.redis;

import java.net.UnknownHostException;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisOperations;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;

@Configuration
@ConditionalOnClass({RedisOperations.class})
@EnableConfigurationProperties({RedisProperties.class})
@Import({LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class})
public class RedisAutoConfiguration {
    public RedisAutoConfiguration() {
    }

    @Bean
    @ConditionalOnMissingBean(
        name = {"redisTemplate"} // 说明我们可以字节顶一个redisTemplate来替换这个默认值
    )
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        RedisTemplate<Object, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    @ConditionalOnMissingBean // 由于String类型是最常使用的
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}

```

## 整合测试

1. 导入依赖：

   ```xml
   <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-redis</artifactId>
           </dependency>
   ```

2. 配置连接：

   ```properties
   spring.redis.host=127.0.0.1
   spring.redis.port=6379
   ```

3. 测试：

   ```java
   package com.lov3camille.springbootredis;
   
   import javafx.application.Application;
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;
   import org.springframework.data.redis.connection.RedisConnection;
   import org.springframework.data.redis.core.RedisTemplate;
   
   @SpringBootTest(classes = AppTest.class)
   class SpringbootRedisApplicationTests {
   
       @Autowired
       RedisTemplate redisTemplate;
   
       @Test
       void contextLoads() {
   
           redisTemplate.opsForValue();
           redisTemplate.opsForList();
           redisTemplate.opsForHyperLogLog(); // operate different data type
   
           RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
           connection.flushAll();
           connection.flushDb(); // get redis connection object
   
           redisTemplate.opsForValue().set("mykey", "wuyan");
           redisTemplate.opsForValue().get("mykey");
       }
   
   }
   
   ```

4. 序列化配置：

   源码如下：

   ```java
   public class RedisTemplate<K, V> extends RedisAccessor implements RedisOperations<K, V>, BeanClassLoaderAware {
       private boolean enableTransactionSupport = false;
       private boolean exposeConnection = false;
       private boolean initialized = false;
       private boolean enableDefaultSerializer = true;
       @Nullable
       private RedisSerializer<?> defaultSerializer;
       @Nullable
       private ClassLoader classLoader;
       @Nullable
       private RedisSerializer keySerializer = null;
       @Nullable
       private RedisSerializer valueSerializer = null;
       @Nullable
       private RedisSerializer hashKeySerializer = null;
       @Nullable
       private RedisSerializer hashValueSerializer = null;
       private RedisSerializer<String> stringSerializer = RedisSerializer.string();
   ...
       
       
       public void afterPropertiesSet() {
           super.afterPropertiesSet();
           boolean defaultUsed = false;
           if (this.defaultSerializer == null) {
               this.defaultSerializer = new JdkSerializationRedisSerializer(this.classLoader != null ? this.classLoader : this.getClass().getClassLoader());
           }
   
           if (this.enableDefaultSerializer) {
               if (this.keySerializer == null) {
                   this.keySerializer = this.defaultSerializer;
                   defaultUsed = true;
               }
   
               if (this.valueSerializer == null) {
                   this.valueSerializer = this.defaultSerializer;
                   defaultUsed = true;
               }
   
               if (this.hashKeySerializer == null) {
                   this.hashKeySerializer = this.defaultSerializer;
                   defaultUsed = true;
               }
   
               if (this.hashValueSerializer == null) {
                   this.hashValueSerializer = this.defaultSerializer;
                   defaultUsed = true;
               }
           }
   ```

   默认的序列化方式是JDK模式，我们需要使用JSON方式序列化。

## 自定义`RedisTemplete`