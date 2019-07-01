---
title: MySQL数据库开发篇
date: 2018-09-05 21:15:17
tags:
---

## MySQL数据库开发篇

<!--more-->

#### （一）表类型（存储引擎）的选择

1. MySQL存储引擎概述

   插件式存储引擎是MySQL数据库最重要的特性之一，用户可以根据应用的需要选择如何

   存储和索引数据、是否使用事务等。一般情况下默认引擎，可以用default-table-type修改

   查看当前默认存储引擎

   ```mysql
   show variables like 'table_type';
   ```

   查看数据库支持的存储引擎,2种方法

   ```mysql
   1. SHOW ENGINES \G;
   2. SHOW VARIABLES LIKE 'have%';
   ```

2. 各种存储引擎

   **MySAM（默认引擎）、InnoDB、MEMORY、MERGE**

#### （二）索引的使用和设计

**一.设计原则**

搜索的索引列，不一定是索要选择的列，最适合的应该是where 子句中的列

使用唯一索引，索引的列，基数越大，索引的效果越好

使用短索引。如果对字符串进行索引，应该指定一个前缀长度，不要对整个列进行索引，对前10个或者20个字符进行索引能节省大量索引时间，也可能会是查询更快；对于较短的键值，索引的高速缓存中的块能容纳更多的键值

利用最左前缀。

**二.作用**

1. 明显加快数据检索速度
2. 如果要确保表中的多列唯一性，就要使用唯一约束，也就是唯一索引
3. 当使用order by 和 group by 子句时，可以明显减少查询的时间
4. 在表与表之间连接查询时，如果创建了索引列，就可以提高表与表之间的连接速度

**三.索引**

1. 普通索引的创建

   **CREATE INDEX index_name ON tabale_name ( column_list (length))**

   CREATE INDEX:创建索引的关键词

   index_name ：索引的名称

   table_name：表名

   column_list：字段列表

   length:  char,varchar 类型时，length可以小于字段的实际长度，如果字段类型为BLOB 或者 TEXT类型，则需要指定length

   **ALTER TABLE table_name ADD INDEX index_name ( column_list)**

   ALTER TABLE :表示修改表的操作

   table_name : 表的名称

   ADD INDEX :关键词，为增加索引

   index_name ：索引名称

   column_list : 索引的字段列表

2. 唯一索引

   **CREATE UNIQUE INDEX index_name ON tabale_name ( column_list (length))**

   CREATE UNIQUE INDEX : 表示创建唯一索引

3. 主键索引

   **CREATE TABLE testpk(**

     **ID INT NOT NULL, name VARCHAR (20) NOT NULL, PRIMARY KEY( ID)**

   **)**

   PRIMARY KEY(ID) : 表示创建主键

4. 查看索引

   **SHOW INDEX FROM tb_name;**

5. 删除索引

   **DROP INDEX index_name ON table_name**

   也可以使用 ALTER 关键字

   **ALTER TABLE table_name DROP INDEX index_name**

### 优化篇

1. 防止sql注入（SQL Injection）,一般来说用PrepareStatement + Bind +variable

2. 正则表达式MySQL用regexp 提供此功能

   ```mysql
   select 'abcdfd' REGEXP '^a'
   ```

   尝试匹配字符串是否以字符'a' 开始，如果匹配返回1，否则为0

3. RAND()函数与 ORDER BY 子句配合使用能够实现随机抽取样本的功能

4. Group By 的 with ROLLUP 子句能够实现类似OLAP的查询

5. MySQL外键功能仅针对InnoDB存储引擎的表有作用

6. 可以用过慢查询日志定位执行效率较低的SQL语句，用 --log-show-querise[=file_name]选项启动

7. 通过explain 分析SQL语句，然后通过适当的索引来增加效率(select)

8. 目前MySQL中索引的存储类型目前只有两种（Btree 和 HASH）

​       对于复合索引，拥有前缀特性，如果只用单独的字段，则用前面的字段，索引会启用，如果只用后面的，则不        启用。另外对于用like 查询的，'%*' 不会使用索引，而'*%' 则可以使用。   如果like 后面跟的是一个列的名字，那么索引也不会使用。

 **存在索引但不会使用索引**

1. MySQL估计使用索引比全表扫描更慢。

2. 使用 MEMORY/HEAP 表并且where 条件中不使用 ”=“ 进行索引列，heap表只有在 ”=“ 的条件下才会使用索引

3. 用 or 分割开的条件，如何or 前的条件中的列有索引，而后面的列中没有索引

   ​

​

