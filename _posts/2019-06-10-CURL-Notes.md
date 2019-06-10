---
layout: post
title: "CURL Notes"
subtitle: "NetCat Tools Notes"
date: 2019-06-10
author: ppbibo
category: Notes
finished: true
---
[TOC]

### ps 

​		**cURL**是一个利用URL语法在[命令行](https://baike.baidu.com/item/命令行)下工作的文件传输工具，1997年首次发行。它支持文件上传和下载，所以是综合传输工具，但按传统，习惯称cURL为下载工具。



## 0x1 重定向

```bash
curl http://6o9.im 
```

回显：

```bash
<html>
<head><title>301 Moved Permanently</title></head>
<body bgcolor="white">
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>

```

使用 -L 跟随链接重定向

```bash
curl http://6o9.im -L
```



## 0x2 获取 HTTP Header

```bash
curl https://6o9.im -I
```

回显

```bash
HTTP/2 200 
server: GitHub.com
content-type: text/html; charset=utf-8
strict-transport-security: max-age=31556952
last-modified: Sat, 08 Jun 2019 01:31:57 GMT
etag: "5cfb100d-1b4f"
access-control-allow-origin: *
expires: Mon, 10 Jun 2019 13:32:28 GMT
cache-control: max-age=600
x-github-request-id: 7018:3E66:217278:24BB82:5CFE5993
accept-ranges: bytes
date: Mon, 10 Jun 2019 13:22:28 GMT
via: 1.1 varnish
age: 0
x-served-by: cache-hnd18739-HND
x-cache: MISS
x-cache-hits: 0
x-timer: S1560172948.045516,VS0,VE119
vary: Accept-Encoding
x-fastly-request-id: 08b285530437c1d104149c1fd6d0dfba5bad6d9a
content-length: 6991
```

显示 HTTP Header 和 文件内容，使用 -i 选项

```bash
curl https://6o9.im -i
```



## 0x3 自定义 User-Agent

```bash
curl -A "Mozilla/5.0 (Android; Mobile; rv:35.0)" https://6o9.im 
```



## 0x4 自定义 header

```bash
curl -H "Referer: www.example.com" -H "Cookie: JSESSIONID=D0112A5063D938586B659EF8F939BE24" -H "User-Agent: Custom-User-Agent" http://www.example.com 
```



## 0x5 使用 -d 发送 POST 请求

```bash
curl http://login.*****.com/index.jsp -d "UserName=admin&UserPwd=HackerTest&Submit=1"
```

在使用 -d 的情况下，如果省略 -X，则默认为 POST 方式



## 0x6 保存与读取Cookie 

1. -c 保存Cookie 后面参数为 fileName

```bash
curl -c "cookie-save" http://login.xxxx.com/index.jsp 
```



2. -b 读取 Cookie 后面参数为 fileName

```bash'
curl -b "cookie-save" http://login.xxxx.com/index.jsp 
```



## 0x7 请求方式的定义

强制使用 GET 方式 

```bash
curl http://login.*****.com/index.jsp -d "UserName=admin&UserPwd=HackerTest&Submit=1" -X GET
```



## 0x8 使用代理

```bash
curl -x 127.0.0.1:1080 http://www.baidu.com
```





