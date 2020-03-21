---
layout: post
title: Eralng ETS消失
date: 2015-05-13 18:08:18
categories: [Erlang]
---

# 问题1

使用-remsh连接到节点，重新装载配置数据（会创建ETS)，结束后，发现存放配置数据的ETS不见了。

# 原因

ETS虽不会参与GC，但当调用ets:delete(ets_name)，或创建ETS进程结束，ETS会被销毁。结束Erlang Shell会话时，创建ETS的进程结束，所有由它创建的ETS也会消失。

# 问题2

使用-remsh连接到节点，在Erlang Shell 中执行函数装载配置数据，执行一些语句导致有异常（如1/0），也会发现存放配置数据的ETS不见了。

# 原因

Erlang Shell遇到异常，会重新创建一个Shell，相当于重新创建1个进程。原来的进程结束，由它创建的ETS也会消失。

# 解决方法

1. 在长期不会退出的进程创建ETS。

2. 使用 ets:new(ets_name, [named_table,{heir,HP,[]}]). 创建ETS。

     当创建ETS进程结束，进程号位HP的进程继承该ETS。

3. 移交拥有权

    ets:give_away(ets_name, P, []).


参见

[ets,当创建的进程死掉之后,ETS的所有权转移到继任者_五月jks_新浪博客](http://blog.sina.com.cn/s/blog_96b8a15401012rwj.html)

[Eralng ets学习总结 - 积累点滴，成就梦想 - ITeye技术网站](http://diaocow.iteye.com/blog/1768647)       
