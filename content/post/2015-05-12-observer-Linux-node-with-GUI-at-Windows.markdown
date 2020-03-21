---
layout: post
title: 在Windows下使用图形化界面查看Linux节点信息
date: 2015-05-12 21:08:18
categories: [Erlang]
---

# 步骤1 在Linux下启动节点

注意使用long name(xxx@计算机名或ip)形式

erl -name linux@192.168.1.136 -setcookie server

# 步骤2  启动Erlang应用程序

点击 开始 -> 所有程序 -> Erlang OTP 17 -> Erlang

# 步骤3  在Erlang中启动用于观察Erlang系统的图形化界面

    Eshell V6.1  (abort with ^G)
    1> observer:start().
    ok

# 步骤4  给本地节点取名字、设置cookie，开始分布式

点击菜单 Nodes -> Enable distribution，点选“Long name”，Node Name 输入 “win@192.168.1.114”， Secret cookie 输入 “server"。点击 "ok"。

# 步骤5 连接到Linux节点

点击菜单 Nodes -> Connect node，输入 "linux@192.168.1.136"，点击 "ok"。即可在Observer上查看连接上的Linux节点的各种信息。


参见

[erlang在windows下和虚拟机节点通信 - 没有开花的树 - 博客频道 - CSDN.NET](http://blog.csdn.net/mycwq/article/details/24738599)            
