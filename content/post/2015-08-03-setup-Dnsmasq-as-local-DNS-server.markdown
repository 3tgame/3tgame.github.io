---
layout: post
title: 使用Dnsmasq作为内部DNS服务器，用于域名针对内外网返回不同IP
date: 2015-08-03 17:10:18
categories: [DNS]
---
# 需求
内网测试时，需要某个域名指向某台内网机器，在电脑上，可修改host。对于手机等不方面改host的情况，或觉得每个人都要修改本地host不方便，可配置DNS使用内部DNS服务器，访问特定域名返回内网IP。

# 介绍
在Linux下可安装Bind或Dnsmasq作为DNS服务。Dnsmasq比Bind简单，对于内网环境够用，支持泛域名、特定域名使用特定上级DNS、屏蔽特定域名，因此选择使用Dnsmasq。

# 安装
yum install dnsmasq

# 配置
dnsmasq 默认使用 /etc/hosts 和 /etc/resolv.conf。如果不希望dnsmasq共享Linux上这些域名服务器和host配置，可在 /etc/dnsmasq.conf 配置

     resolv-file=/etc/dnsmasq.d/resolv.dnsmasq.conf
     addn-hosts=/etc/dnsmasq.d/dnsmasq.hosts

/etc/dnsmasq.d/resolv.dnsmasq.conf 配置上级DNS服务器

     nameserver 192.168.1.1
     
/etc/dnsmasq.d/dnsmasq.hosts 配置域名解析

     192.168.1.168 test.com

如上配置，解析 a.test.com 还是 使用原来的域名记录，而非指向192.168.1.168，要想子域名生效，可在 /etc/dnsmasq.conf 配置泛解析

     address=/test.com/192.168.1.168

配置特定域名使用特定DNS服务器

     server=/google.com/8.8.4.4

更改完后要重启dnsmasq服务：service dnsmasq restart

配置防火墙开放53端口

     vi /etc/sysconfig/iptables
     -A INPUT -p udp -m state --state NEW --dport 53 -j ACCEPT
     -A INPUT -p tcp -m state --state NEW --dport 53 -j ACCEPT

# 测试
在客户端配置dns服务器为 安装dnsmasq的服务器的ip。
查看 nslookup xxx.test.com 的输出，确认配置生效。
如没有该命令，使用 yum install bind-utils 安装

参看 [CentOS6.5 64bit如何安装DNS服务dnsmasq 作为翻墙利器_技术奇客_ITGeeker](http://itgeeker.net/centos6-5-64bit-how-to-install-dnsmasq-broken-gwf/)
