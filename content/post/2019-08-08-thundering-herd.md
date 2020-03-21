---
layout: post
title: 惊群
date: 2019-08-08 18:10:18
categories: [Linux]
tags: ["Linux"]
---
惊群现象（thundering herd）就是当多个进程和线程在同时阻塞等待同一个事件时，如果这个事件发生，会唤醒所有的进程，但最终只可能有一个进程/线程对该事件进行处理，其他进程/线程会在处理失败后重新休眠，这种性能浪费就是惊群。  
# accept()惊群
  主线程创建socket，bind()，listen()后，调用fork()创建多个子进程，在子进程调用accept()，当有客户端连接进来，多个子进程的accept()都会返回，但只有一个子进程的accept()会返回成功，其他子进程的accept()都会返回失败的结果。  
  在Linux2.6之后，accept不会引起惊群。内核收到1个客户端连接后，只会唤醒等待队列中第一个进程。
# epoll()惊群
  主进程创建socket，bind()，listen()，把该socket加到epoll后，调用fork()创建多个子进程，在子进程调用epoll_wait()，当有客户端连接进来，多个子进程的epoll_wait()会返回，但只有一个子进程的epoll_wait()会返回成功，其他子进程的epoll_wait()都会返回失败的结果：EAGAIN（Resource temporarily unavailable）。  
  并不是所有进程都会被唤醒，当某个进程处理完这个IO事件，内核没有这个事件，就不会唤醒其他进程，所以被唤醒的子进程的数量是不固定的。如果在epoll_wait()返回后，且处理事件前，加入代码sleep(2)，则可看到所有子进程都会被唤醒。  
## 为什么内核处理accept()的惊群，但不处理epoll()的惊群？  
  accept()只能被一个进程处理。但epoll监听的文件描述符，除了可能被accept()，可能还包括其他网络IO事件。其他网络IO事件是否只由一个进程处理，就不确定。
# 线程惊群
  pthread_cond_broadcast()会唤醒队列上所有线程去处理事件，但只有一个线程会真正获得事件的控制权
# 解决惊群的方法
1. 在accept前先加锁，获得锁的才accept。Nginx的解决方法：在同一时刻，只有一个nginx worker进程把监听socket加到自己的epoll中。  
2. Linux内核的3.9版本带来了SO_REUSEPORT特性，该特性支持多个进程或者线程（多个socket）绑定到同一IP、端口。  
    优点：在服务器套接字上没有锁的竞争，提高服务器程序的性能。  
        在内核层面实现对多个socket的负载均衡：所有reuseport同一个IP地址/端口的套接字会挂在一个链表上，当有连接到来时，用数据包的源IP/源端口作为一个HASH函数的输入，将结果对reuseport套接字数量取模，得到一个索引，该索引指示的数组位置对应的套接字便是工作套接字。  
    缺点：由于缺少一致性哈希，当listen socket的数目发生变化（比如新服务上线、已存在服务终止）的时候，根据SO_REUSEPORT的路由算法，在客户端和服务端正在进行三次握手的阶段，最终的ACK可能不能正确送达到对应的socket，导致客户端连接发生Connection Reset，所以有些请求会握手失败。  
3. liunx 4.5内核在epoll已经新增了EPOLL_EXCLUSIVE选项，在多个进程同时监听同一个socket，只有一个被唤醒。

参考：[惊群效应 - 云+社区 - 腾讯云](https://cloud.tencent.com/developer/article/1340628)  
[Linux惊群效应详解（最详细的了吧） - lyztyycode的博客 - CSDN博客](https://blog.csdn.net/lyztyycode/article/details/78648798)  
[accept与epoll惊群 - 纯真年代](https://pureage.info/2015/12/22/thundering-herd.html)