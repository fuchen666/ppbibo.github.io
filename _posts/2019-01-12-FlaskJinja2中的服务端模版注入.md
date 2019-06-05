---
layout: post
title: "SSTI的服务端模版注入"
subtitle: "Flask && Jinja2中的服务端模版注入"
date: 2019-01-12
author: ppbibo
category: security
finished: true
---
[TOC]

## Flask/Jinja2中的服务端模版注入

如果你还没听说过SSTI(服务端模版注入)，或者对其还不够了解，在此之前建议去阅读一下James Kettle写的一篇[文章](http://blog.portswigger.net/2015/08/server-side-template-injection.html)问题出在开发者定义一个404错误页面，该开发者选择使用字符串格式化将URL动态添加到模版字符串中并返回到页面中，用户对模板是可控的。



## 正文

定义一个错误页面

定义了一个模板字符串

使用格式化字符串的方式将URL动态添加到模版字符串中并返回在页面上

用户对模板是可控的

![img](/static/img/a0.png) 

![img](/static/img/a1.png) 

### 触发一个xss

```bash

http://127.0.0.1:5000/<script>alert(/xss/)</script>

```

![img](/static/img/a2.png) 

 

 参考：

http://www.freebuf.com/articles/web/98619.html
