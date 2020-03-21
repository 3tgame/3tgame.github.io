---
layout: post
title: cannot load hipe，不能启动使用rebar generate产生的release
date: 2014-12-15 16:50:18
categories: [Erlang]
---

HiPE是High-Performance Erlang的缩写，HiPE使用即时编译技术可提高Erlang程序的运行效率。使用 rebar generate 发布应用后，使用 xxx/bin/xxx console 启动应用（其中xxx为应用名），会提示错误：

{"init terminating in do_boot",{'cannot load',hipe_amd64_liveness,get_files}}

进入erlang shell，

{% highlight erlang %}
1> hipe:version().
** exception error: undefined function hipe:version/0
{% endhighlight %}

原来是Centos上安装的Erlang环境没有HiPE。

# 解决方法1：把HiPE从release排除

在 reltool.config 增加 {app, hipe, [{incl_cond, exclude}]}

重新执行 rebar generate

# 解决方法2：安装HiPE

[Download Erlang OTP | Erlang Solutions](https://www.erlang-solutions.com/downloads/download-erlang-otp) 中Centos安装指引最后有说明：

sudo yum install erlang-hipe

重新执行 rebar generate
