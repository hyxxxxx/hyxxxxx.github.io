---
layout: post
title: "投票排行业务设计思路"
date: 2017-07-13
excerpt: "不知不觉间还是做了好几个投票系统，感觉主要业务万变不离其中，所以单独把主要业务分享出来"
tags: 
- Original
- Redis
- Database
comments: true
---


### 前言
投票的目的其实是为了支持排行

所以这两个功能要联系起来设计

由于票数这个字段会不断增长，如果用关系型数据库来做，会**频繁插入或者修改**，用户量只要稍微一大，就会非常低效，无疑是不妥的。

REDIS不仅高效快速，而且提供**ZSET**这种有序集合，当然就用它了

### 业务场景
下面给定一个业务场景来进行分析：

每位选手都有自己所属的参赛地，以市为最小单位，上面还有省份，所以排行榜有三个维度分别是：**市排行**、**省排行**和**总排行**

### 实现思路
我们先从排行榜功能开始分析
1. 首先我们把选手ID塞进ZSET来记录投票并排名，定义KEY为：`VOTE_RANK_SET`

        每一个维度都定义一个标识，C代表市、P代表省、F代表所有
        如果某城市的ID是520，就有一个ZSET集合的KEY为`VOTE_RANK_SET:C520`
        每个城市和省份的KEY都依次定义，集合的值就是该区域下的选手ID
        
2. 其次需要把排行列表所需要的选手信息存入HASH，定义KEY为：`VOTE_ENTITY_HASH`

		如果选手ID是666，那就存在一个KEY为`VOTE_ENTITY_HASH:666`
		
3. 这样，投票的时候就应该把该选手所在的集合的票数同时增加

4. 查询排名时，只需要把对应集合的选手ID拿出来，以ID去查询HASH就行了

### 代码实现
多说无益，还是直接贴代码吧

保存选手HASH
`stringRedisTemplate.boundHashOps(key).putAll();`

给选手投票，选手ID=1，所在城市ID=100，所在省份ID=1001
这样我们就有三个KEY分别是：
```java
String key1 = "VOTE_RANK_SET:C100";
String key2 = "VOTE_RANK_SET:P1001";
String key3 = "VOTE_RANK_SET:F0";		//这个代表全部
stringRedisTemplate.opsForZSet().incrementScore(key1, 选手ID, 票数);
stringRedisTemplate.opsForZSet().incrementScore(key2, 选手ID, 票数);
stringRedisTemplate.opsForZSet().incrementScore(key3, 选手ID, 票数);
```
**这个KEY应该是根据选手ID查询出所在的区域来动态拼接的**

最后就是查询排行榜了

`stringRedisTemplate.boundZSetOps`提供多种方法来操作ZSET

下面列举几个我们可能会使用到的接口：
- 下面这两个方法是一回事，size调用zCard，就是返回该key的元素总数
`Long size();`
`Long zCard();`
- 获取元素的分数，这里元素就是选手ID，结果就是票数
`Double score(Object o);`
- 获取给定分数段内的元素总数
`Long count(double min, double max);`
- 第一个方法是按照元素下标的区间查询，第二个是按照得分的区间查询，返回值就是VALUE的集合
`Set<V> reverseRange(long start, long end);`
`Set<V> reverseRangeByScore(double min, double max);`
- 这两个方法的作用跟上面差不多，不同在于返回值是个对象集合，从这个对象中，我们可以取到该元素的VALUE和SCORE，这个还是比较有用的
`Set<TypedTuple<V>> reverseRangeWithScores(long start, long end);`
`Set<TypedTuple<V>> reverseRangeByScoreWithScores(double min, double max);`
	
    **reverse的意思是反转，意味着取出来的顺序是倒序的，这正是排行榜需要的顺序**

现在取出该区域下的选手ID后，就要依次去查询选手的HASH列表了，这里要使用REDIS的管道技术，使这段操作可以高效稳定的执行

stringRedisTemplate同样提供多种开启管道的方法，这里就不进行API的介绍了

总之，把HASH查询出来并封装成对象，包括统计这个选手的票数什么的操作都放在管道中处理就好

大致逻辑如下：
```java
//分页查询选手ID
Set<TypedTuple<String>> ids = stringRedisTemplate.opsForZSet()
    .reverseRangeByScoreWithScores(KEY, Integer.MIN_VALUE, Integer.MAX_VALUE, pageNum, pageSize);

return stringRedisTemplate.execute((RedisCallback<List<T>>) connection -> {
    List<T> result = new ArrayList<>();
    connection.openPipeline();		//开启管道
    //循环选手ID的SET集合，
    for (TypedTuple<String> id : ids) {
        Map<Object, Object> map = stringRedisTemplate.opsForHash().entries(KEY);
        //封装成对象<T>加入结果集
        result.add(T);
    }
    connection.closePipeline();		//关闭管道
    return result;		//返回集合
 });
```
