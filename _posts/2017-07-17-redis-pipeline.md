---
layout: post
title: "Redis管道技术应用"
date: 2017-07-17
excerpt: "主要介绍Java调用Redis管道相关API"
tags: 
- Original
- Redis
- Database
- Java
comments: true
---

### 什么是REDIS管道技术
因为REDIS是一种基于客户端-服务端模型以及请求/响应协议的TCP服务，所以客户端会阻塞等待服务端的返回结果。

然而REDIS管道技术可以在服务端未响应时，客户端可以继续向服务端发送请求，并最终一次性读取所有服务端的响应。

这样的结果是什么呢？就是，快！

### 如何使用
这里介绍的是spring-redis中的API
```java
<T> T execute(RedisCallback<T> action);
<T> T execute(SessionCallback<T> session);
List<Object> executePipelined(RedisCallback<?> action);
List<Object> executePipelined(final RedisCallback<?> action, final RedisSerializer<?> resultSerializer);
List<Object> executePipelined(final SessionCallback<?> session);
List<Object> executePipelined(final SessionCallback<?> session, final RedisSerializer<?> resultSerializer);
<T> T execute(RedisScript<T> script, List<K> keys, Object... args);
<T> T execute(RedisScript<T> script, RedisSerializer<?> argsSerializer, RedisSerializer<T> resultSerializer,
			List<K> keys, Object... args);
```
可以发现spring-redis提供了很多相关方法，以execute命名的方法不罕见，在JDK中线程池相关操作也是execute，所以这类方法的共性都是执行一段逻辑罢了。

使用execute方法的好处是**不用频繁去找连接池要链接**，可以使用**一个链接处理一段逻辑**，所以如果有一段逻辑中**多次操作**REDIS的，就最好采用这类方法。

接下来分别介绍下它们的用法

在介绍方法用途之前，有这么几个参数类型是它们共有的，有必要先了解下这几个参数是什么意思
1. RedisCallback
		这个接口需要实现doInRedis方法，参数是RedisConnection
		可以利用RedisConnection调用一些REDIS最底层的方法，基本和REDIS命令一个等级了
		这个方法不需要关心连接关闭和异常处理问题
2. SessionCallback
		这个接口需要实现execute方法，参数是RedisOperations
		可以利用RedisOperations执行一系列操作
		可以用事务相关命令控制事务
3. RedisSerializer
		序列化和反序列化实现
		REDIS不接收空值参数，但是可以返回空结果（比如不存在的KEY）
4. RedisScript
		REDIS2.6版本后可以支持脚本

### 方法介绍
- `<T> T execute(RedisCallback<T> action);`

	返回值可由自己定义，自动适配DAO实现，自动选择序列化实现，**不支持事务**，不用关心连接声明周期，**可以开启管道**

	调用栗子：
    ```java
    String str = redisTemplate.execute((RedisCallback<String>) connection -> {
            connection.openPipeline();		//开启管道
            connection.closePipeline();		//关闭管道
            byte[] bytes = connection.get("".getBytes());		//可以操作数据
            return Arrays.toString(bytes);
    });
    ```


-  `<T> T execute(SessionCallback<T> session);`

	在同一个会话中执行多种REIDS操作，**支持事务**，**不能使用管道**

	调用栗子：
    ```java
    redisTemplate.execute(new SessionCallback<String>() {
            @Override
            public <K, V> String execute(RedisOperations<K, V> operations) throws DataAccessException {
                operations.watch();		//可以事务管理
                // Read
                operations.multi();
                // Write operations
                operations.exec();
                return null;
            }
    });
    ```

- `List<Object> executePipelined(RedisCallback<?> action);`

	使用方法同上，只不过这类方法会**自动开启管道**，REDIS的管道技术是**不支持事务**的，并且该方法不能指定返回值，如果定义了返回值会抛异常：
    ```java
    Object result = action.doInRedis(connection);
    if (result != null) {
            throw new InvalidDataAccessApiUsageException(
                    "Callback cannot return a non-null value as it gets overwritten by the pipeline");
    }
    ```
    所以返回值不能自定义，而是根据管道执行完毕后所查询出来的结果，每条以Object对象形式装进List返回。

	调用栗子：
    ```java
    List<Object> objects = redisTemplate.executePipelined(new SessionCallback<Object>() {
            @Override
            public <K, V> Object execute(RedisOperations<K, V> operations) throws DataAccessException {
                    operations.opsForValue().get("TEST_KEY");
                    operations.opsForValue().get("ORG_SERVICE:472");
                    return null;
            }
    });
    ```
