---
layout: post
title: Socket 被动模式和delay_send结合使用的注意点
date: 2015-05-16 18:18:18
categories: [Erlang]
---

# 问题

使用init:stop()方式结束节点的时候就不能正常结束

# 出现条件

启动服务器后，没有客户端连接上来，马上使用init:stop()，可以正常结束节点。
如有客户端连接上来，断开连接，会使用delay_send向刚断开的Socket，发送错误信息。使用init:stop()，不能正常结束节点。

# 原因

Socket选项设置如下：

    -define(TCP_OPTIONS,
        [binary, {packet, 0}, {active, false}, {reuseaddr, true}, {nodelay, false},
        {delay_send, true}, {send_timeout, 5000}, {keepalive, true}, {exit_on_close, true}]).

在被动模式下，若Socket对端关闭，本端不会terminate port，如果在这之后还向该Socket发送数据，则因为设置的是delay_send，所以不会调用sock_sendv()真正发送数据，而是存到发送队列里。最后在该Socket宿主进程退出时，会terminate port，但发现发送队列不为空，所以就将该port设为ERTS_PORT_SFLG_CLOSING。

节点内如果有port状态为ERTS_PORT_SFLG_CLOSING，使用init:stop()就不能关闭节点。

更多 源码、说明 参看[关于erlang socket被动模式和delay_send合用的问题 - Skyman的部落格 - 博客频道 - CSDN.NET](http://blog.csdn.net/skymanwu/article/details/8739248)。

# 解决方法

不要在对端关闭后（如收到{inet_async, _ClientSocket, _Ref, {error, closed}}后）再向该Socket发送数据；或者关掉delay_send；或者用非被动模式。


参见

[关于erlang socket被动模式和delay_send合用的问题 - Skyman的部落格 - 博客频道 - CSDN.NET](http://blog.csdn.net/skymanwu/article/details/8739248)

[Erlang socket passive mode and delay_send of combined - Programmer Share](http://www.programmershare.com/1827715/)

[gen_tcp的close与delay_send交叉问题 - - ITeye技术网站](http://wqtn22.iteye.com/blog/1765741)

[gen_tcp调用进程收到{empty_out_q, Port}消息奇怪行为分析  系统技术非业余研究](http://blog.yufeng.info/archives/1489)
