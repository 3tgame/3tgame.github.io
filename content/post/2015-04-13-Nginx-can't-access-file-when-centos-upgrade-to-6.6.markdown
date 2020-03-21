---
layout: post
title: Centos升级到6.6，请求网页，Nginx提示原来没有的错误
date: 2015-04-13 14:30:18
categories: [Nginx]
---

# 问题1

请求 http://debug.wxhl.qijian.com/pay.html，Nginx错误日志提示 

==> /var/log/nginx/error.log <==
2015/04/13 15:39:10 [error] 10704#0: *28 open() "/data/wxhl/web/debug/pay.html" failed (13: Permission denied), client: 192.168.1.114, server: debug.wxhl.qijian.com, request: "GET /pay.html HTTP/1.1", host: "debug.wxhl.qijian.com"

==> /var/log/audit/audit.log <==
type=AVC msg=audit(1428910750.812:14880): avc:  denied  { read } for  pid=10704 comm="nginx" name="pay.html" dev=dm-2 ino=9832981 scontext=unconfined_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:admin_home_t:s0 tclass=file
type=SYSCALL msg=audit(1428910750.812:14880): arch=c000003e syscall=2 success=no exit=-13 a0=14d6864 a1=800 a2=0 a3=7f37b7a569e0 items=0 ppid=10703 pid=10704 auid=0 uid=497 gid=497 euid=497 suid=497 fsuid=497 egid=497 sgid=497 fsgid=497 tty=(none) ses=1451 comm="nginx" exe="/usr/sbin/nginx" subj=unconfined_u:system_r:httpd_t:s0 key=(null)

使用以下命令 查看权限信息
     $ namei -om /data/wxhl/web/debug/pay.html
f: /data/wxhl/web/debug/pay.html
dr-xr-xr-x root root /
lrwxrwxrwx root root data -> /home/data
   dr-xr-xr-x root root /
   drwxr-xr-x root root home
   drwxr-xr-x root root data
drwxr-xr-x root root wxhl
drwxr-xr-x root root web
drwxrwxrwx root root debug
-rwxrwxrwx root root pay.html
父目录的权限都有 可执行 权限，pay.html 有可读权限。权限没问题。

# 问题2

请求 http://debug.wxhl.qijian.com/info.php，Nginx错误日志提示

==> /var/log/nginx/error.log <==
2015/04/13 15:34:20 [error] 10704#0: *18 FastCGI sent in stderr: "Primary script unknown" while reading response header from upstream, client: 192.168.1.114, server: debug.wxhl.qijian.com, request: "GET /info.php HTTP/1.1", upstream: "fastcgi://127.0.0.1:9000", host: "debug.wxhl.qijian.com"

==> /var/log/audit/audit.log <==
type=AVC msg=audit(1428910460.317:14879): avc:  denied  { getattr } for  pid=10683 comm="php-fpm" path="/home/data/wxhl/web/debug/info.php" dev=dm-2 ino=9831008 scontext=unconfined_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:home_root_t:s0 tclass=file
type=SYSCALL msg=audit(1428910460.317:14879): arch=c000003e syscall=6 success=no exit=-13 a0=7fff5b214190 a1=7fff5b214080 a2=7fff5b214080 a3=1d items=0 ppid=10678 pid=10683 auid=0 uid=497 gid=497 euid=497 suid=497 fsuid=497 egid=497 sgid=497 fsgid=497 tty=(none) ses=1451 comm="php-fpm" exe="/usr/sbin/php-fpm" subj=unconfined_u:system_r:httpd_t:s0 key=(null)

查看 Nginx 关于虚拟空间的配置：
server {
    listen       80;
    server_name  debug.wxhl.qijian.com;
  
    location ~ \.php$ {                                                                                                                                                                             
        root           /data/wxhl/web/debug;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }  
} 

配置正确，且 文件 /data/wxhl/web/debug/info.php 确实存在，权限都正确。

# 原因

按照
[How to Solve "No input file specified" with PHP and Nginx](http://blog.martinfjordvald.com/2011/01/no-input-file-specified-with-php-and-nginx/) 里介绍的方法进行检查，还是没解决。

后来重装Nginx，请求HTML页面提示 Open File Permission denied，从
[centos - Nginx 403 forbidden for all files - Stack Overflow](http://stackoverflow.com/questions/6795350/nginx-403-forbidden-for-all-files)
开始怀疑是SELinux的问题，检查相关目录、文件权限都正确，访问路径也正确，从 audit.log 可确定 是 SELinux 限制访问 的原因。使用 getenforce 查看 SELinux 当前的模式为：Enforcing

# 解决方法1

audit2why 可用于解析 audit.log 中关于安全政策的信息，可使用 yum install policycoreutils-python 安装 audit2why。

更多 查看 [SE Linux changes when upgrading to RHEL 6.6 / CentOS 6.6](http://nginx.com/blog/nginx-se-linux-changes-upgrading-rhel-6-6/)
              

# 解决方法2

使用 setforce 0，关掉SELinux
