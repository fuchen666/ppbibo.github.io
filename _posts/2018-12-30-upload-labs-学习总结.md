---
layout: post
title: "upload-labs 学习总结"
subtitle: "upload-labs summary"
date: 2018-12-30
author: ppbibo
category: Notes
finished: true
---
[TOC]

   在web应用程序中，上传的功能是一种常见的需求，例如上传头像，上传视频或上传其他类型的文件，如果未对上传的文件进行严格的验证和过滤，就容易造成上传任意文件，包括上传一个木马.



[![0](./upload-labs 总结 _ bibo @ 安全小生_files/0.png)](/static/img/0.png)

## 0x1 客户端javascript检测（通常为检测文件扩展名）

```bash
1. 禁用js 

2. F12 修改 js 验证代码（例如: 将php的格式添加进去）

3. BurpSuite 抓包改名
```

## 0x2 服务端MIME类型检测（检测Content-Type内容）

MIME的作用 ： 使客户端软件，区分不同种类的数据，例如web浏览器就是通过MIME类型来判断文件是GIF图片，还是可打印的PostScript文件。

web服务器使用MIME来说明发送数据的种类， web客户端使用MIME来说明希望接收到的数据种类。

Tomcat的安装目录\conf\web.xml 中就定义了大量MIME类型 ，你可也去看一下。

MIME的作用 ： 使客户端软件，区分不同种类的数据，例如web浏览器就是通过MIME类型来判断文件是GIF图片，还是可打印的PostScript文件。

```bash
绕过方法：

直接使用burp抓包，得到post上传数据后，将 Content-Type: text/plain 改成 Content-Type: image/gif

就可以成功绕过。
```

## 0x3 可重写文件解析规则绕过

.htaccess 文件攻击 配合名单列表绕过，上传一个自定义的.htaccess，就可以轻松绕过各种检测

通过一个.htaccess 文件调用 php 的解析器去解析一个文件名中只要包含”haha”这个字符串的 任意文件，所以无论文件名是什么样子，只要包含”haha”这个字符串，都可以被以 php 的方式来解析，是不是相当邪恶，一个自定义的.htaccess 文件就可以以各种各样的方式去绕过很 多上传验证机制

建一个.htaccess 文件，里面的内容如下

```bash
### 包含 "haha" 这个字符串，都可以被以 php 的方式来解析
<FilesMatch "haha">
SetHandler application/x-httpd-php 
</FilesMatch>

-----------------------------------
### 包含 ".jpg" 这个字符串，都可以被以 php 的方式来解析
<FilesMatch ".jpg">
SetHandler application/x-httpd-php
</FilesMatch>
```

同目录有个我们上传一个只有文件名并包含字符串”haha”，但是却无任何扩展名的文件 里面的内容是 php 一句话木马。

## 0x4 服务端检测绕过(目录路径检测)

上传的文件名写成`11.jpg`, save_path改成`../upload/11.php%00`，最后保存下来的文件就是`11.php`

```bash
目录路径检测，一般就检测路径是否合法，但稍微特殊一点的都没有防御。 比如比较新的 fckeditor php <= 2.6.4 任意文件上传漏洞

当 POST 下面的 URL 的时候

/fckeditor264/filemanager/connectors/php/connector.php?Command=FileUpload&Type=Image&

CurrentFolder=fuck.php%00.gif HTTP/1.0

CurrentFolder 这个变量的值会传到 ServerMapFolder($resourceType, $folderPath, $sCommand) 中的形参 $folder 里，而 $folder 在这个函数中并没做任何检测，就被 CombinePaths()了
```

## 0x5 可被利用过滤

这种只删除一次php的，即可：
双写文件名绕过，文件名改成`hacker.pphphp`

## 0x6 文件头检查

```bash
添加GIF图片的文件头 GIF89a，绕过GIF图片检查。
```

## 0x7 后缀名黑名单

##### 黑名单

1. 文件名后缀大小写混合绕过

```bash
.php 改成 .pHP 然后上传即可
```

1. 名单列表绕过

```bash
用黑名单里没有的名单进行攻击，比如黑名单里没有

语言	        可解析后缀
asp/aspx	asp,aspx,asa,asax,ascx,ashx,asmx,cer,aSp,aSpx,aSa,aSax,aScx,aShx,aSmx,cEr
php	        php,php5,php4,php3,php2,pHp,pHp5,pHp4,pHp3,pHp2,html,htm,phtml,pht,Html,Htm,pHtml
jsp		jsp,jspa,jspx,jsw,jsv,jspf,jtml,jSp,jSpx,jSpa,jSw,jSv,jSpf,jHtml
```

1. 可被利用Windows系统的特性

```bash
1. 比如发送的 http 包里把文件名改成 test.asp. 或 test.asp_ (下划线为空格)，这种命名方式 在 windows 系统里是不被允许的，所以需要在 burp 之类里进行修改，然后绕过验证后，会被 windows 系统自动去掉后面的点和空格，但要注意 Unix/Linux 系统没有这个特性。

2. Windows文件流特性绕过
文件名改成 .php::$DATA , 上传成功后保存的文件名其实是 .php
```

1. 0x00 截断绕过

```bash
在扩展名检测这一块目前我只遇到过 asp 的程序有这种漏洞，给个简单的伪代码 

name = getname(http request) //假如这时候获取到的文件名是 test.asp .jpg(asp 后面为 0x00)
type = gettype(name) //而在 gettype()函数里处理方式是从后往前扫描扩展名，所以判断为 jpg
if (type == jpg)
	....
```

1. .htaccess 文件攻击 配合名单列表绕过，上传一个自定义的.htaccess，就可以轻松绕过各种检测

##### 白名单

1. 0x00 截断绕过

```bash
用像 test.asp%00.jpg 的方式进行截断，属于白名单文件，再利用服务端代码的检测逻辑 漏洞进行攻击，目前我只遇到过 asp 的程序有这种漏洞
```

1. 解析调用/漏洞绕过 这类漏洞直接配合上传一个代码注入过的白名单文件即可，再利用解析调用/漏洞

```bash
# 文件解析漏洞 :解析漏洞主要说的是一些特殊文件被iis、apache、nginx在某种情况下解释成脚本文件格式的漏洞

IIS5.x/6.0解析漏洞：

1.文件解析：

	.asp;.jpg

第二种，在IIS6.0下。分号后面的不被解析，也就1.asp;.jpg

2.目录解析

	.x.asp/1.jpg

	./liuwx.asa

	/liuwx.cer

	/liuwx.cdx

3.asa，cer，（都是以asp格式来运行）
Apache解析漏洞：

Apache是从右边到左开始判断解析，如果为不可识别，就继续往左判断

可以用1.php.xxx
IIS 7.0/IIS7.5/Nginx<8.03畸形解析漏洞：

1.jpg/x.php	 ,会以php脚本执行

x.asp%00.jpg 

a.aspx.a;.a.aspx.jpg..jpg
```

## 总结：

 通过上传漏洞可以直接拿到一个webshell ，其危害性也是显而易见的，通过上传漏洞我们可以直接拿到web应用程序的权限进而提权拿到服务器管理员权限执行系统命令内网漫游等。

## 修复：

 1. 上传的路径要限制在固定路径下

 2. 上传文件路径只给只读和写权限，不需要执行权限

 3. 服务端文件类型使用白名单过滤，后台不应有添加扩展名类型功能

 4. 文件上传使用自己的命名规则重新命名上传的文件（防止解析漏洞）

[构造优质上传漏洞fuzz字典](http://gv7.me/articles/2018/make-upload-vul-fuzz-dic/)