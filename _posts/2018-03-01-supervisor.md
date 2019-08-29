---
layout: post
title: "supervisor 笔记"
subtitle: "supervisor 笔记"
date: 2018-12-18
author: ppbibo
category: Notes
finished: true
---
[TOC]

##  supervisor

```bash
yum -y install  epel-release,supervisor,vim
mkdir /etc/supervisord.d/
echo_supervisord_conf > /etc/supervisord.conf
```

```bash
$ vim /etc/supervisord.conf

[program:flask]
directory = /root/Cert-Auditing-project ; 程序的启动目录
command = python3 flask_web.py
autostart = true     ; 在 supervisord 启动的时候也自动启动
startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正>常启动了
autorestart = true   ; 程序异常退出后自动重启
startretries = 3     ; 启动失败自动重试次数，默认是 3
user = root          ; 用哪个用户启动
redirect_stderr = true  ; 把 stderr 重定向到 stdout，默认 false
stdout_logfile_maxbytes = 20MB  ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 20     ; stdout 日志文件备份数
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所
以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile = /var/logs/usercenter_stdout.log
```

将supervisor配置为开机自启动服务

```bash
$ vim /usr/lib/systemd/system/supervisord.service

[Unit]
Description=Supervisor daemon

[Service]
Type=forking
PIDFile=/var/run/supervisord.pid
ExecStart=/bin/supervisord -c /etc/supervisord.conf
ExecStop=/bin/supervisorctl shutdown
ExecReload=/bin/supervisorctl reload
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target

```

```bash
$ systemctl enable supervisord
```







