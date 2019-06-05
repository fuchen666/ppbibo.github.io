---
layout: post
title: "Safety test ideas"
subtitle: "Safety test ideas"
date: 2018-12-28
author: ppbibo
category: security
finished: true
---
[TOC]

## 渗透测试思路整理

### PS:

记录下渗透测试的思路最近考试脑子感觉不是自己的了

知识面,决定看到的攻击面有多广 , 知识链,决定发动的杀伤链有多深。



[![1](/static/img/1.jpg)](/static/img/1.jpg)

#### 0x1 前期信息收集

1. 确定渗透测试的范围及目的性
2. 判断网站的类型
3. 了解公司业务及web服务的功能
4. 收集公司(个人)信息
5. 中间件版本的查询（apache，iis，nginx，tomcat 等）
6. Google Hacker
7. 敏感目录扫描( 御剑 / .git / .svn /[www.rar等敏感目录](http://www.xn--rar-vj3f12f0vjis8a1jh/))
8. 查询CMS
9. 端口扫描
10. 二级/三级 域名（扩展渗透的范围）
11. C 段检测

#### 0x2 每一步的目的性

##### 1. 判断网站的类型

编程语言可以测试index.{php,asp,aspx,jsp,do,action}也可以在返回头信息看到。

以 .do .action 都可以测试下Struts2漏洞

确定网站类型来梳理渗透思路

##### 2. 了解公司业务及web服务的功能

```
1.是否有搜索框能否注入或XSS
2.是否有留言框能否XSS打cookie
3.有登录尝试登录框是否有sql注入或XSS
4.是否有验证码
	验证码能否被识别
	验证码能否被无视
	
5.注册是否有sql漏洞
6.是否能注册管理权限的账号
7.验证手机的情况下
	是否能无限轰炸
	是否能任意更改
	验证码是否有时间验证
	验证码是否相同
	是否能爆破验证码

8.登录是否能越权
	未授权访问
	SQL注册测试
	
9.个人信息是否有存储型XSS
10.个人信息能否CSRF
	有验证机制能否绕过

11.个人信息是否存在越权
	改变uid等敏感参数检测是否越权
	平行越权
	垂直越权
	
12.头像上传是否能getshell
	直接getshell
	解析漏洞
	上传漏洞
	是否能上传txt

13.修改密码能否CSRF
	有验证机制能否绕过

14.找回密码是手机号码验证
	是否能无限轰炸
	是否能任意更改
	验证码是否有时间验证
	验证码是否相同
	是否能爆破验证码

15.是否有购买商品功能
	任意修改支付金额
	任意修改商品数量
	收货地址存储型XSS

16.配置检查
17.API接口安全
18.逻辑错误漏洞
```

##### 3. 收集公司(个人)信息

社会工程学

```
whois 查询
   
注册人姓名
   
注册人常用ID
   
注册人QQ
   
注册人邮箱
   
社工库(旧密码)
   
(组合密码 + top1000)
   
前端源代码注释

Github等开放平台泄漏敏感信息
   
DNS记录域名商
   
.....
```

##### 4. 中间件版本的查询（apache，iis，nginx，tomcat 等）

```
中间件：
(iis一定支持asp，asa,cer)(apache常见就是php)(tomcat,jboss,weblogic,jetty为常见的JAVA程序中间件)
   
iis在低版本常见的漏洞有：
    解析漏洞 IIS6.0（asp/aspx）
    文件解析漏洞：文件名为*.asp;.jsp
    目录解析漏洞：文件夹名为*.asp 那么这个文件名下面的所有文件就会解析为asp文件
    IIS6.0 默认可执行的还有*.asa *.cer *.cdx                                

IIS7.0/IIS7.5/Nginx<8.03 畸形解析漏洞
    正常后缀加/*.php 如：xxx.com/upload/1.jpg/1.php
    *.php.123.234 或 *.php.123;*.php.jpg..jps 以此类推下去直到解析为止(php,asp,aspx)（适量）
    建议也多试几次iis6.0的解析漏洞，有些时也是可行的。
Apache在低版本常见的漏洞有：
   	解析漏洞
    1.从右往左判断解析 如：*.php.rar.jpg.png 等等把常见后缀都写上去直到解析为止（适量）
    2.*.php 改为*.php1,,*.php2,*.php3,*.php4 以此类推下去直到解析为止（适量）
```

##### 5. Google Hacker

```
site 允许你搜索仅仅位于一个特定服务器上的或者在一个特定域名里的页面
filetype 搜索特定后缀的文件
link 包含指定网页的链接的网页
inanchor 寻找链接的锚点
cache 显示页面的缓存版本
numberange 搜索一个数字 例如：numberange:12344-12346
daterange 搜索在特定日期范围内发布的页
....

举例：
site:xxxxx.com
intitle:index.of site:xxxxx.com
intitle:login 
intext:版权信息
inurl:php?id=
filetype:mdb inurl:xxxxx.com
link:xxxxx.com
filetype:log inurl:log # 日志查找
...
```

[Google Hacking信息刺探的攻与防](https://www.freebuf.com/articles/network/169601.html)

[谷歌黑客数据库](https://www.exploit-db.com/google-hacking-database)

##### 6. 敏感目录扫描

查找敏感信息，网站后台，网站源码或数据库的备份文件，上传地址等信息可以更近一步的进行渗透测试。

例如：

```
/robots.txt
/admin.php
/admin_login.php
/admin/login.php
/user_login.php
/upload.php
/up2.php
/phpmyadmin
/dede
/www.rar
/wwwroot.rar
/备份.rar
/*.sql
/.git
/.svn

...
```

Funning PHP

```
# 本地测试的是5.5以下版本都可以

/?=PHPB8B5F2A0-3C92-11d3-A3A9-4C7B08C10000   (PHP信息列表)
/?=PHPE9568F34-D428-11d2-A769-00AA001ACF42   (PHP的LOGO)
/?=PHPE9568F35-D428-11d2-A769-00AA001ACF42   (Zend LOGO)
/?=PHPE9568F36-D428-11d2-A769-00AA001ACF42   (PHP LOGO 蓝色大象)

## 官方声明这不是一个安全漏洞，你可以通过下面的方法关闭
	# 在 php.ini中设置 expose_php = Off
：
```

[参考](https://bugs.php.net/)

##### 7. 查询的CMS

确定网站的CMS及版本信息可以查找0day进行攻击，如果开源可以进行白盒测试，否则只能黑盒测试了。

```
门户：
	地方门户：DZ，phpcms ...
	小型企业门户：xiaocms ...

论坛：
	国内论坛基本都是：dede，DZ，winphp，帝国 ...

博客：
	wordpress，Zblog，typecho，emlog ...

商场：
	ecshop ...
```

##### 8. 端口扫描

使用Nmap 对服务器进行端口扫描，查询其开放的服务。

```
21 ftp   
	弱口令,爆破
	匿名访问
	
22 SSH
	弱口令,爆破
	按28次Backspace键漏洞
	OpenSSH用户枚举漏洞（CVE-2018-15473）

23 Telnet 
	弱口令登录

137,139  Smaba服务
	弱口令,爆破
	未授权访问
	远程代码执行漏洞(CVE-2015-0240)

389 LDAP  轻量目录访问协议
	爆破，弱口令
	
443 SSL 心脏滴血/HTTPS
	OpenSSL心脏滴血漏洞(CVE-2014-0160)
	
445 
1080 socket 代理
1433 MSSQL
	弱口令,爆破
	
1521 Oracle
	弱口令,爆破
	
3306 MySQL
	弱口令,爆破
	
5432 PostgreSQL
	弱口令,爆破
	
6379 redis
	未授权访问
	弱口令
	
27017 Mongodb
	弱口令,爆破
	
5000 DB2
	弱口令,爆破

7001 WebLogic
	弱口令(weblogic,weblogic|system,weblogic或weblogic123|portaladmin,guest)
	爆破
    后台部署war包getshell
    java反序列 
    SSRF
    
2601 zebra路由
	弱口令,默认密码zebra
	
3389 远程桌面  
	弱口令,爆破
	shift 后门
	
8080 tomcat
	弱口令

2181  Zookeeper
	未授权访问
	echo envi |nc 127.0.0.1 2181   # 输出关于服务环境的详细信息(区别于 conf 命令)
	echo conf | nc 127.0.0.1 2181  # 输出相关服务配置的详细信息
	echo stat|nc 127.0.0.1 2181    # 来查看哪个节点被选择作为follower或者leader
 	echo ruok|nc 127.0.0.1 2181    # 测试是否启动了该Server，若回复imok表示已经启动
	echo dump| nc 127.0.0.1 2181   # 列出未经处理的会话和临时节点
	echo kill | nc 127.0.0.1 2181  # 关掉server
	echo cons | nc 127.0.0.1 2181  # 列出所有连接到服务器的客户端的完全的连接 / 会话的详细信息
	echo reqs | nc 127.0.0.1 2181  # 列出未经处理的请求
	echo wchs | nc 127.0.0.1 2181  # 列出服务器 watch 的详细信息
	echo wchc | nc 127.0.0.1 2181 # 通过 session 列出服务器 watch 的详细信息，它的输出是一个与 watch 相关的会话的列表
	echo wchp | nc 127.0.0.1 2181 # 通过路径列出服务器 watch 的详细信息。它输出一个与 session 相关的路径
 
11211 memcache
	未授权访问

......
```

[CVE-2018-15473](https://www.freebuf.com/vuls/184583.html)

[CVE-2015-0240](https://www.freebuf.com/vuls/59898.html)

[CVE-2014-0160](https://segmentfault.com/a/1190000000461002)

##### 9. 域名爆破

```
二级/三级 域名（扩展渗透的范围）
```

##### 10. C 段检测

```
扩展渗透的范围
```

#### 0x3 渗透测试策略

经厂商授权的渗透测试

1. 黑盒测试
2. 白盒测试

```
1. 技术侦查 -> DNS侦查 -> 子域名枚举 -> 端口扫描 -> 目录枚举 -> cms侦查 -> Google Hacker
2. 社会工程学 -> 弱口令top1000 -> 数据泄露 -> Whois查询 -> 邮箱收集 -> 公开仓库分析(Github等) -> 代码注释 -> {邮件|网站|?}钓鱼
3. 分析建模 -> 建立草稿 -> 整理思路
4. 公开漏洞匹配 -> 服务器漏洞匹配 -> web漏洞匹配
5. 漏洞挖掘分析 -> 测试范围{服务器|pc客户端|网站|微信服务号|微信小程序|Android应用...}
6. 进阶测试 -> pass
7. 证据搜集
8. 风险评估 -> 漏洞分级 -> 漏洞详情介绍 -> 漏洞参考
9. 生成报告
```

#### 0x4 安全防护

```
漏洞自查，生成报告日志保存最少六个月
日志的保存方法（ 权限控制 && 转存第三方 ）
资产整理
漏洞扫描器(周期性扫描)
定期代码审查
关注敏感函数的使用
关注整体的逻辑
关注权限的检查
WAF{悬镜|安全狗|D盾|云锁}
报警监控
```