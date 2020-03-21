---
published: true
layout: post
title: Linux使用QQ企业邮箱服务器发送邮件
date: 2017-03-28T14:18:18.000Z
categories:
  - email centos
---
发送邮件可通过sentmail或postfix服务转发邮件，但如果只需要发送邮件，没必要再搭建邮件服务。直接使用外部STMP服务发送邮件。本文分别介绍通过mailx和ssmtp这2种客户端使用QQ企业邮箱STMP服务发送邮件。

# 方法1：使用mailx
## 安装
     yum install mailx

## 配置
编辑 /etc/mail.rc 文件，添加如下内容：

     set from=develop@xxx.com
     set smtp=smtp.exmail.qq.com
     set smtp-auth-user=develop@xxx.com
     set smtp-auth-password=xxx
     set smtp-auth=login
## 测试发送邮件
     echo "This will go into the body of the mail." | mail -v -s "Hello world" develop@xxx.com

# 方法2：使用ssmtp
## 安装
     yum install ssmtp

## 配置
编辑 /etc/ssmtp/ssmtp.conf 文件，输入以下内容：
     
     # 在日志中输出详细的调试信息
     #Debug=YES
     # 发送给user id < 1000的所有信件都发送给这个账号
     root=develop@xxx.com
     # QQ企业邮箱服务器，如果使用TLS，则使用端口465，否则使用端口25
     mailhub=smtp.exmail.qq.com:465
     # 在QQ企业邮箱的账号
     Hostname=develop@xxx.com
     # “From：”字段信息使用客户端信件提供的
     FromLineOverride=YES
     # 使用TLS协议发送消息
     UseTLS=YES
     #IMPORTANT: The following line is mandatory for TLS authentication
     TLS_CA_File=/etc/pki/tls/certs/ca-bundle.crt   
     # 在QQ企业邮箱的账号
     AuthUser=develop@xxx.com
     # 邮箱密码
     AuthPass=xxxxx

为了使默认的“From”字段（root）使用邮箱账号，编辑 /etc/ssmtp/revaliases 文件，输入以下内容：

     root:develop@xxx.com:smtp.exmail.qq.com

## 测试发送邮件
     echo -e "Subject: Hello world\n\nThis will go into the body of the mail. " |  ssmtp -vvv develop@xxx.com

## 设置ssmtp为默认的mta，用于发送邮件
如想mail命令默认使用ssmtp发送邮件，执行以下命令，选择ssmtp：

     alternatives --config mta

如没有mail命令，先执行 yum install mailx。

从 /var/log/maillog 可查看日志信息。

# 参见
[How To Setup Email Alerts on Linux Using Gmail or SMTP](https://www.howtogeek.com/51819/how-to-setup-email-alerts-on-linux-using-gmail/)
 
[LINUX下通过外部SMTP发邮件 （直接抛弃sendmail和postfix） - xiaoshi1991 - 博客园](http://www.cnblogs.com/xiaoshi1991/archive/2012/09/19/2694465.html)
