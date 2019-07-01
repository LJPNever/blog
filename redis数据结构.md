---
title: redis 的数据结构
date: 2018-11-11 09:19:58
tags:
---

1. ### Redis 的数据结构

   <!--more-->

   Redis 是一个基于键值对（key-value）的非关系型数据库，Redis 数据库中的每个键值对都是由对象组成的：

   ​    数据库键总是一个字符串对象（string object）；

      数据库的值可以为字符串对象(string)、列表对象（list）、哈希对象（hash）、集合对象（set）、  有序集合（ zset）对象这五种对象的其中一种。这些数据类型都支持push/pop、

   ​    add/remove以及取交集并集及更丰富的操作，而且这些操作都是原子性的

   简单操作：

   字符串操作（string）

   set key value  

   get key  

   del key

    ![a114f3147daeb638620f9f40d752f8e](https://neverljp-1256310950.cos.ap-guangzhou.myqcloud.com/a114f3147daeb638620f9f40d752f8e.png)

   列表操作（list）

   lpush 将元素推入列表的左端

   rpush 将元素推入列表的右端

   lpop  从列表左端弹出元素

   rpop  从列表右端弹出元素

   lindex 获取列表在给定位置上的一个元素

   lrange 获取列表在给定范围上的所有元素

   lpushx 只能插入已经存在的key,且一次只能插入一次

   ![e0a9b993309d39241585bf9d3193dfe](https://neverljp-1256310950.cos.ap-guangzhou.myqcloud.com/e0a9b993309d39241585bf9d3193dfe.png)	

   ​						![df1b8ec4f2cb232015852d25683834b](https://neverljp-1256310950.cos.ap-guangzhou.myqcloud.com/df1b8ec4f2cb232015852d25683834b.png)

   集合操作（set）

   SADD将元素添加到集合     成功添加返回1，如果返回0则表示集合中已经有这个元素了

   SREM从集合里面移除元素     存在返回1，不存在返回0

   SISMEMBER快速地检查一个元素是否已经存在于集合中

   SMEMBERS获取集合包含的所有元素

   ​

   ![e9066065767673b794d3cfe107df92c](https://neverljp-1256310950.cos.ap-guangzhou.myqcloud.com/e9066065767673b794d3cfe107df92c.png)	

   哈希操作（hash）

   HSET     在散列里面关联起给定的键值对

   HGET     获取指定散列键的值

   HGETALL     获取散列包含的所有键值对

   HDEL     如果给定键存在于散列里面，那么移除这个键

   ![b2b25d7d88d66f764f2d17c9bea4b51](https://neverljp-1256310950.cos.ap-guangzhou.myqcloud.com/b2b25d7d88d66f764f2d17c9bea4b51.png)	![e959e69ee5d3bba5a4c1204b2f5f1f6](https://neverljp-1256310950.cos.ap-guangzhou.myqcloud.com/e959e69ee5d3bba5a4c1204b2f5f1f6.png)

    有序集合（zset）

   有序集合的键被成为成员，每个成员都是各不相同的。有序集合的值被成为分值，分值必须为浮点数。
   有序集合是redis里面唯一一个既可以根据成员访问元素，又可以根据分值以及分值的排列顺序来访问元素 的结构。
   ZADD     将一个带有给定分值的成员添加到有序集合里面
   ZRANGE     根据元素在有序排列中所处的位置，从有序集合里面获取多个元素
   ZRANGEBYSCORE     获取有序集合在给定分值范围内的所有元素

   ​							![f38331f0fa9b3bec8a45e31a2bc35d1](https://neverljp-1256310950.cos.ap-guangzhou.myqcloud.com/f38331f0fa9b3bec8a45e31a2bc35d1.png)

   ​

   ![7b96d927f91e256b50e9752086eefcc](https://neverljp-1256310950.cos.ap-guangzhou.myqcloud.com/7b96d927f91e256b50e9752086eefcc.png)


