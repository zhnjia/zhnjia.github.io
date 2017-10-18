---

layout: default
title: Setup A Systemd Service

---


# Table of Contents

1.  [添加配置文件](#orga7b2468)
2.  [任务管理](#orge6af97c)

添加一项需要长期在后台运行的任务，可以选择将其添加到系统服务。下面将以shadowsocks为例演示如何使用systemd添加一项服务。

参考链接：<http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html>


<a id="orga7b2468"></a>

# 添加配置文件

    $ cat /lib/systemd/system/shadowsocks.service
    [Unit]
    Description=Shadow Socks
    After=network-online.target
    [Service]
    Type=simple
    ExecStart=/usr/local/bin/sslocal -c /etc/ss.json
    ExecStop=/bin/kill -9 $MAINPID
    [Install]
    WantedBy=multi-user.target
    WantedBy=graphical.target

简单解释下：

> [Unit]
> After: 何时启动
> [Service]
> Execstart: 任务启动命令
> Execstop: 如何结束任务
> [Install]
> Wantedby: 与其他服务的依赖关系


<a id="orge6af97c"></a>

# 任务管理


## 激活

    $ sudo systemctl enable shadowsocks


## 启动

    $ sudo systemctl start shadowsocks


## 停止

    $ sudo systemctl stop shadowsocks


## 状态

    $ systemctl status shadowsocks


## 更新配置文件

    $ sudo systemctl daemon-reload

修改配置文件后，需要reload才能刷新到system
