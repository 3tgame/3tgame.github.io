---
layout: post
title: 使用 wireshark 和 tcpdump 实时抓取Android的包
date: 2015-01-06 11:10:18
categories: [Android]
---

# 下载可运行在Android上的tcpdump

从 [USB Sniffing with tcpdump - OMAPpedia](http://omappedia.org/wiki/USB_Sniffing_with_tcpdump) 下载编译好的tcpdump，
[Pre-compiled binaries](http://omappedia.org/wiki/File:Tcpdump-4.3.0-arm.tar.gz)

# 上传tcpdump到Android机器上

adb push tcpdump /data/local/tcpdump

如提示没有权限，则先root。

# 安装netcat

Android安装应用busybox，安装完后会有nc命令

# 使用adb shell 在 Android 设备上执行

{% highlight sh %}
> adb shell
$ su
# /data/local/tcpdump -n -s 0 -w - | busybox nc -l -p 11233 
{% endhighlight %}

在11233端口上建立服务，使用 tcpdump 抓包作为输入。

# 在电脑上执行

{% highlight sh %}
adb forward tcp:11233 tcp:11233 && d:\install\ncat.exe 127.0.0.1 11233 | "D:\Program Files\Wireshark\Wireshark.exe" -k -S -i -
{% endhighlight %}

以上命令使用 adb froward 把到本地的11233端口的socket连接转送到 Android上的11233端口，使用 ncat 建立1个客户端，连接到本地 127.0.0.1 的11233端口，得到的输出作为Wireshark的输入。

Windows下没有nc命令，可到 [Ncat主页](http://nmap.org/ncat/)下载[已编译好的Windows版本](http://nmap.org/dist/ncat-portable-5.59BETA1.zip)


上面2步相当于把在Android上使用tcpdump抓的包用Wireshark显示

参见：

[实时监控Android设备网络封包-UC技术博客](http://tech.uc.cn/?p=2278)

[如何利用tcpdump抓取andorid网络数据请求，Wireshark可清晰的查看到网络请求的各个过程包括三次握手，但Fiddler进行网络数据抓包和展现更方便](http://www.trinea.cn/android/tcpdump_wireshark/)

[9.2. 从命令行启动Wireshark](http://man.lupaworld.com/content/network/wireshark/c9.2.html)
