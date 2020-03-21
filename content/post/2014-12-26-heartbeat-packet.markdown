---
layout: post
title: 心跳包
date: 2014-12-26 17:10:18
categories: [Socket]
---

一方突然关掉电源、防火墙切断不活跃连接、网络出现问题，都可导致一方掉线，另一端如没有发送数据，不会知道该情况。

# 有2种方法处理上述问题

1、TCP keepalive

2、应用层实现心跳包

# TCP 有keepalive的规范，对于简单场景可使用 TCP keepalive。

1、检测已失去连接的对端。

2、保持链路存活（keepalive）

# 既然有keepalive，为什么还要有心跳包？

心跳解决的也是 TCP Keepalive要解决的问题，只是更灵活，可在应用层做断线处理，无视传输层是使用TCP 还是 UDP。

参见：

[服务端为什么需要心跳(保活)机制 - 推酷](http://www.tuicool.com/articles/Q3M73q)

[TCP Keepalive HOWTO](http://www.tldp.org/HOWTO/html_single/TCP-Keepalive-HOWTO/#preventingdisconnection)

[从微信谈起，如何优化互联网APP心跳机制](http://tech.sina.com.cn/t/csj/2013-04-24/09288273431.shtml)

[【Socket】关于socket长连接的心跳包 - 成鹏致远 - 博客园](http://www.cnblogs.com/lcw/p/3565459.html)
