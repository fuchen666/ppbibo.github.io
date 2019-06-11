---
layout: post
title: "NMAP Notes" 
subtitle: "NMAP Notes"
date: 2018-12-26
author: ppbibo
category: Notes
---



[TOC]

读《web安全攻防渗透测试实战指南》



##### 正文

```
-iL 从txt导入目标IP进行扫描 nmap -iL iplist.txt
--exclude 后面跟不扫描的IP 例子：nmap -iL iplist.txt --exclude 127.0.0.1
--excludefile 从文件中导入不扫描的ip 例子： nmap -iL iplist.txt --excludefile notip.txt
-sL 仅列举指定目标IP，不进行主机扫描 例子：nmap -sL 127.0.0.1 192.168.3.39|nmap -iL iplist.txt -sL
-sn 只进行主机发现，不进行端口扫描 例子：nmap -sn 192.168.3.39
-Pn 将所有指定的主机视为开启，不进行主机发现 例子:nmap -Pn 192.168.3.39
-PS/PA/PU/PY 使用TCP/SYN/ACK或SCTP INIT/ECHO 方式来发现 例子：nmap -PS 192.168.3.39|nmap -PA 192.168.3.39|nmap -PU 192.168.3.39|nmap -PY 192.168.3.39
-PE/-PP/PM 使用ICMP echo，timestamep,netmask请求包发现主机 例子：nmap -PE 192.168.3.39|nmap -PP 192.168.3.39|nmap -PM 192.168.3.39
-PO 使用IP协议包探测对方主机是否开启 例子：nmap -PO 192.168.3.39
-n/-R -n表示不进行DNS解析：-R表示总是进行DNS解析 例子：nmap -sV -n 127.0.0.1|nmap -R 127.0.0.1
--dns-servers 指定DNS服务器 例子：nmap -R 192.168.3.39 --dns-servers
--system-dns 指定使用系统的DNS服务器 例子：nmap -R 192.168.3.39 --system-dns
--traceroute 追踪每个路由节点 例子：nmap 192.168.3.34 --traceroute
-sS/sT/sA/sW/sM 指定使用：TCP SYN/Connect()/ACK/Window/Maimon scans的方式对目标主机进行扫描
-sU 使用UDP方式扫描UDP端口 例子：nmap -sU 192.168.3.39
-sN/sF/sX 指定使用TCP Null/FIN/Xmas scans秘密扫描的方式协助探测对方的TCP端口状态
--scanflags 定制TCP包flags 例子：nmap -sV 192.168.3.39 --scanflags 1
-sI 指定使用Idle scan的方式扫描目标主机 例子：nmap -sI 空闲主机IP -Pn 目标主机IP
-sY/sZ 使用SCTP INIT/COOKIE-ECHO扫描SCTP协议端口的开放状态
-sO 确定目标主机支持的协议 例子：nmap -sO 192.168.3.39
-b 使用FTP bounce scan扫描方式 例子：nmap -b 192.168.3.39
-p  指定扫描的端口 例子：nmap -p 3389,445 192.168.3.39
-F 快速扫描模式，只扫描TOP100的端口 例子：nmap -F 192.168.3.39
-r 不进行端口打乱的操作，让nmap的扫描不轻易的被对方的防火墙检测到 例子：nmap -p 445,135 -r 192.168.3.39

更多脚本请参考：https://nmap.org/nsedoc/categories
--top-ports 扫描开放概率最高的端口 （Nmap作者曾经做过大规模的扫描，以统计开放概率最高的端口。默认情况下nmap会扫描最有可能的1000个端口） 例子：nmap 192.168.3.39 --top-ports 445
--port-raito 扫描指定频率以上的端口
-sV 指定nmap进行版本探测 例子：nmap -sV 192.168.3.39
--version-intensity 指定版本探测深度(范围:0-9) nmap -sV 192.168.3.39 --version-inensity 9
--version-light 指定轻量级侦探方式（intensity 2）
--version-all 尝试使用所有的probes进行侦测
--version-trace 显示出详细版本侦测过程信息
-A 表示全方面扫描 例子：nmap -A 192.168.3.39
-T 扫描速度 (范围：1-5) 例子 nmap -A-T5 192.168.3.39
-sC 全面收集信息 例子：nmap -sC 192.168.3.39
-sP 判断目标是否存活
-O 判断目标操作系统
-sF 探测目标防火墙状态
常用的扫描方式:
nmap 192.168.3.39
nmap 192.168.3.0/24 扫描网段
nmap -A -T5 192.168.3.35
nmap -sV -T5 192.168.3.39
nmap -sC 192.168.3.39
nmap -sS 192.168.3.39
nmap -sC -n -sV --version-inensity 9 -T5 192.168.3.39
nmap -iL iplist.txt --exclude 127.0.0.1
nmap -iL iplist.txt --excludefile notip.txt
nmap -p 135,445,3389 192.168.3.39
nmap --traceroute 192.168.3.39
nmap -O 192.168.3.39
nmap -sF 192.168.3.39

常见6种nmap端口状态及协议
open 开放的
filetered 被过滤的，表示端口被防火墙或其他网络设备阻止，不能访问
close 关闭的
unfiltered 未被过滤的，表示nmap无法确定端口所在状态,需进一步探测
open/filetered 开放的或被过滤的，nmap不能识别
close/filetered 关闭的或被过滤的，nmap不能识别


Nmap进阶：
nmap的默认脚本存放在/xx/nmap/scripts的文件夹下
nmap的脚本主要分为以下几类
Auth 负责处理鉴权处理
Broadcast 在局域网内探查更多服务的开启情况，如DHCP/DNS/SQLserver
Brute 针对常见的应用提供暴力破解方式，如HTTP/SMTP等
Default 针对-sC或-A选项提供默认脚本，提供基本脚本的扫描能力
Discovery 对网络进行更多的扫描，如SMB枚举，SNMP查询等
Dos 用于进行拒绝服务攻击
Exploit 利用已知的漏洞入侵系统
External 利用第三方数据库或资源，例如：进行whois解析
Fuzzer 模糊测试脚本，发送异常包给目标机，探测潜在漏洞
Intrusive 入侵性脚本，此类脚本可能会引起对方的IDS/IPS的记录等
Malware 探测目标机是否感染了病毒，开启了后门信息等
Safe 此类脚本与Intrusive相反，属于安全脚本
Version 负责增强服务与版本扫描功能的脚本
Vuln 负责检查目标机是否有常见的漏洞如MS17-010

脚本使用方法 例子：nmap --script=auth（脚本名称） IP
常用脚本：
用户还可以根据配置--script=类别进行扫描，常用参数如下表示
-sC/--script=Default 使用默认的脚本进行扫描
--script=<Lua scripts> 使用某个脚本进行扫描
--script-args=key1=value1,key2=value2 该参数用于传递脚本里的参数，key1是参数名，该参数对应value1这个值。如果还有更多参数逗号连接
--script-args-file=filename 使用文件为脚本提供参数
--script-trace 如果设置参数，则显示脚本执行过程中发送与接收数据
--script-updatedb 在nmap的scripts目录中有一个script.db文件，该文件保存了当前nmap可用的脚本,类似于一个小型的数据库，如果我们开启nmap并调用了此参数，就会进行脚本更新
--script-help 输出对应脚本的使用参数，以及详细的介绍信息 用法：nmap --script-help auth
```

[转载九世](https://422926799.github.io/2018/12/21/nmap-note/)