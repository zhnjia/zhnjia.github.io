---

layout: page
title: SSH tunnel

---


# Table of Contents

1.  [拓扑图](#orgff1a873)
2.  [配置](#org948265a)
3.  [其他](#org771a324)

SSH隧道提供了一种通过公网电脑，利用SSH反向代理，使得位于两个不同局域网或内网的电脑可以互通的方式。

下面以从家到办公室的连接为例，说明下如何建立SSH隧道。


<a id="orgff1a873"></a>

# 拓扑图

![img]({{"/assets/sshtunnel.png" | absolute_url}})


<a id="org948265a"></a>

# 配置


## Cloud

新建一个用户“incloud”，确保在Home或者Office可以使用该用户ssh登录Cloud。


## Office

1.  将本地用户itsme的key上传至Cloud（这步可以不做，只是省得每次登录要输入密码）
2.  安装autossh, 并配置成系统服务：

        $ cat /lib/systemd/system/autossh.service
        [Unit]
        Description=Auto SSH Tunnel
        After=network-online.target
        [Service]
        User=itsme
        Type=simple
        ExecStart=/usr/bin/autossh -p 22 -M 60323 -NR '*:60322:0.0.0.0:22' incloud@45.45.45.45
        ExecReload=/bin/kill -HUP $MAINPID
        KillMode=process
        Restart=always
        [Install]
        WantedBy=multi-user.target
        WantedBy=graphical.target
3.  修改ssh\_config, 添加如下两行，确保ssh连接稳定：

        $ tail /etc/ssh/ssh_config -n2
            ServerAliveInterval 10
            ServerAliveCountMax 100
4.  激活并启动autossh.service


## Home

使用如下命令可以直接ssh穿透到Office：

    $ ssh -p 60322 itsme@45.45.45.45


<a id="org771a324"></a>

# 其他

1.  启动autossh.service后，检查隧道是否建立成功，尤其是遇到“Connection refused”错误是，要查看Cloud端的端口：

        $ sudo netstat -tunlp
        Active Internet connections (only servers)
        Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
        tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      852/sshd
        tcp        0      0 0.0.0.0:60322           0.0.0.0:*               LISTEN      8390/sshd: incloud
        tcp        0      0 0.0.0.0:60323           0.0.0.0:*               LISTEN      8390/sshd: incloud
        udp        0      0 127.0.0.1:123           0.0.0.0:*                           790/ntpd
        udp        0      0 0.0.0.0:123             0.0.0.0:*                           790/ntpd
        udp6       0      0 :::123                  :::*                                790/ntpd

    -   看到60322和60323这两个端口对应的Local Address是0.0.0.0，就表示隧道已经打通；如果是127.0.0.1, 那会报"Connecting refused"!
    -   如果Local Address显示的是127.0.0.1:60322, Debian系列的发行版请添加"GatewayPorts yes"到/etc/ssh/sshd\_config, 其他的添加到/etc/ssh/ssh\_config.
2.  如果使用阿里云之类的服务，记得修改“安全组”，把60322/60323这两个端口添加到白名单。
