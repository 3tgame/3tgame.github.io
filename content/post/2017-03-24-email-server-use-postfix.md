---
published: true
layout: post
title: 使用Postfix搭建邮件服务器
date: 2017-03-24T17:18:18.000Z
categories:
  - email centos
---
Linux下搭建邮件服务器，包括用于发送邮件的 Postfix（Postfix 比 Sendmail容易配置、处理更快），用于接收邮件的 Dovecot，用于使用web页面操作邮件的 Squirrelmail。下文只说明搭建用于发送邮件的Postfix。


# 为邮件服务器添加DNS解析
虽然不加DNS解析也能把邮件发出去，但会被大多数邮件服务器当作垃圾邮件。需要添加三条DNS解析记录：A记录、MX记录、TXT记录，指向邮件服务器所在机器。A记录和MX记录是必须的，填写TXT记录提升发送外域邮件的成功率。

TXT记录用来保存域名的附加文本信息，TXT记录的内容按照一定的格式编写，最常用的是SPF格式，SPF用于登记某个域名拥有的用来外发邮件的所有ip地址。MX记录的作用是给寄信者指明某个域名的邮件服务器有哪些，SPF格式的TXT记录的作用跟MX记录相反，它向收信者表明，哪些邮件服务器是经过某个域名认可发送邮件的。SPF的作用主要是反垃圾邮件，主要针对那些发信人伪造域名的垃圾邮件。按照SPF格式在DNS中增加一条TXT类型的记录，将提高该域名的信誉度，同时可以防止垃圾邮件伪造该域的发件人发送垃圾邮件。最典型的spf格式的txt记录例子为“v=spf1 a mx ~all”，表示只有这个域名的a记录和mx记录中的ip地址有权限使用这个域名发送邮件。

# 安装
     yum install postfix

# 配置
编辑 /etc/postfix/main.cf 文件，内容如下：

     # 邮件服务器的主机名称，若信件的mail to 字段是该主机名称，且该名称符合mydestination的设定，信件会被该主机收下，否则进行relay判断。
     myhostname = mail.xxx.com
     # 主机名称去掉第一部分后的域名。
     mydomain = xxx.com
     # 邮件地址@后面的内容，寄信时没有加上mail from字段时，使用这个值。
     myorigin = $myhostname
     # Postfix系统监听的网络接口。
     inet_interfaces = all
     # Postfix的通讯协定。
     inet_protocols = ipv4
     # 能够接收信件的主机名称，mail to字段的值是以下主机名称，才会被这台机器接收。
     mydestination = $myhostname, $mydomain, localhost.$mydomain, localhost
     # 信任的用户端，才能进行relay（转递信件到其他邮件服务器）。以下配置信任 本机，局域网内和特定配置的用户。
     mynetworks = 127.0.0.0/8, 192.168.1.0/24, hash:/etc/postfix/access
     # 可以relay到下一台MTA主机，以下配置可以转递邮件到 $mydestiantion，qq.com和126.com的邮件服务器。
     relay_domains = $mydestination, qq.com, 126,com
     # 邮件别名路径
     alias_maps = hash:/etc/aliases
     # 邮件别名资料库路径
     alias_database = hash:/etc/aliases

## 邮件服务器的使用权限 /etc/postfix/access
/etc/postfix/access 用于控制relay的用户，一下配置允许 120.114.141.60 和 .edu.cn 域名的用户使用这台机器转递信件，而不允许 192.168.2.0/24 和 ban.com 域名的用户：

     120.114.141.60      OK
     .edu.cn                   OK
     ban.com                 REJECT
     192.168.2.              REJECT

使用这个文件控制的好处是不用重启Postfix，只要执行以下命令即可，会生成 /etc/postfix/access.db（特定格式，加快读取配置）。

     postmap hash:/etc/postfix/access

## 邮件别名 /etc/aliases
系统中有很多系统账号，如apache、mysql、nginx等，以这些账号执行的程序有消息产生时，将会以email方式传给谁，因为这些系统账号没有密码登陆，看不到邮件，所以这些系统账号的消息都发送给root。这就是通过 /etc/alias 设置的，内容如下：

     mailer-daemon: postmaster
     postmaster: root
     bin: root
     daemon: root

左边是别名，右边是实际存在的账号或email address。

也可以用于用于发送群邮件，如发送邮件到student2011这个不存在的账号，邮件会被发送到各个账号，/etc/aliases 配置如下：

     student2011: std001,std002,std003,std004

执行以下命令，生成邮件别名资料库  /etc/aliases.db：

     postalias hash:/etc/aliases

如果对外开放访问邮件服务器，则需要开放25端口：

     iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 25 -j ACCEPT

# 启动
     service postfix start
     
开机启动

     chkconfig --level 345 postfix on

# 寄信测试：

## 方法1：使用mail工具
     echo "This will go into the body of the mail." | mail -s "Hello world" xxx@qq.com

## 方法2：使用telnet
     telnet localhost smtp

     Trying ::1...
     Connected to localhost.
     Escape character is '^]'.
     220 server.unixmen.local ESMTP Postfix
     ehlo localhost                             ## type this command ##
     250-server.unixmen.local
     250-PIPELINING
     250-SIZE 10240000
     250-VRFY
     250-ETRN
     250-ENHANCEDSTATUSCODES
     250-8BITMIME
     250 DSN
     mail from:root                             ## Type this - mail sender address##
     250 2.1.0 Ok
     rcpt to:xxx.qq.com                         ## Type this - mail receiver address ##
     250 2.1.5 Ok
     data                                       ## Type this to input email message ##
     354 End data with <CR><LF>.<CR><LF>
     welcome to unixmen mail server             ## Enter the boddy of the email ##
     .                                          ## type dot (.) to complete message ##
     250 2.0.0 Ok: queued as B822221522
     quit                                       ## type this to quit from mail ##
     221 2.0.0 Bye
     Connection closed by foreign host.
    
可从日志文件 /var/log/maillog 查看错误信息，排查原因。

# 其他
## 问题1
使用以下命令发送邮件，提示“send-mail: Cannot open mail:25”

     echo "This will go into the body of the mail." | mail -s "Hello world" xxx@qq.com

原因：错误提示 表示打开到服务器mail的25端口，也是因为没有修改mail 配置

解决：修改 mail 客户端配置文件 /etc/ssmtp/ssmtp.conf 如下：

     mailhub=localhost
 
## 问题2
执行  postmap hash:/etc/postfix/access ，提示“postmap: /usr/local/mysql/lib/libmysqlclient.so.18: no version information available (required by postmap) ”

原因：执行 locate mysqlclient |xargs ls -lha ，输出

     lrwxrwxrwx  1 root root    20 3月  21 14:39 /usr/lib64/mysql/libmysqlclient_r.so.18 -> libmysqlclient.so.18
     lrwxrwxrwx  1 root root    24 3月  21 14:39 /usr/lib64/mysql/libmysqlclient_r.so.18.1.0 -> libmysqlclient.so.18.1.0
     lrwxrwxrwx  1 root root    24 3月  21 14:39 /usr/lib64/mysql/libmysqlclient.so.18 -> libmysqlclient.so.18.1.0
     -rwxr-xr-x  1 root root  9.2M 11月 29 07:12 /usr/lib64/mysql/libmysqlclient.so.18.1.0
     lrwxrwxrwx. 1 root mysql   17 10月 15 2015 /usr/local/mysql/lib/libmysqlclient_r.so -> libmysqlclient.so
     lrwxrwxrwx. 1 root mysql   20 10月 15 2015 /usr/local/mysql/lib/libmysqlclient_r.so.18 -> libmysqlclient.so.18
     lrwxrwxrwx. 1 root mysql   24 10月 15 2015 /usr/local/mysql/lib/libmysqlclient_r.so.18.0.0 -> libmysqlclient.so.18.0.0
     lrwxrwxrwx. 1 root mysql   20 10月 15 2015 /usr/local/mysql/lib/libmysqlclient.so -> libmysqlclient.so.18
     lrwxrwxrwx. 1 root mysql   24 10月 15 2015 /usr/local/mysql/lib/libmysqlclient.so.18 -> libmysqlclient.so.18.0.0
     -rwxr-xr-x  1 root mysql 6.9M 10月 15 2015 /usr/local/mysql/lib/libmysqlclient.so.18.0.0

因为 postmap 需要的libmysqlclient版本与 使用libmysqlclient 不一致。

解决：编辑 /etc/ld.so.conf，把 /usr/local/mysql/lib/mysql 加到最前面，执行 ldconfig。

# 参见
[鳥哥的 Linux 私房菜 -- Mail Server](http://linux.vbird.org/linux_server/0380mail.php#postfix)

[How To Install Postfix on CentOS 6 -- DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-postfix-on-centos-6)

[Setup Local Mail Server Using Postfix, Dovecot And Squirrelmail On CentOS 6.5/6.4 -- Unixmen](https://www.unixmen.com/install-postfix-mail-server-with-dovecot-and-squirrelmail-on-centos-6-4/)
