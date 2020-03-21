---
published: true
layout: post
title: 同一台机器部署多个Tomcat实例
date: 2017-04-11T16:18:18.000Z
categories:
  - tomcat centos
---
如果需要在同一台机器上安装多个Tomcat，如给不同开发人员使用，或部署开发、测试、生产环境、或者用于测试负载均衡，可在一台机器上部署多个Tomcat实例。

简单的方式，可以拷贝整个Tomcat目录到新的目录，修改 server.xml中监听端口的配置，就可得到一个新的Tomcat实例。

Tomcat支持以更轻量的方式配置多实例。Tomcat有2个环境变量，CATALINA_HOME 指向Tomcat二进制分发版本的根目录。CATALINA_BASE 指向Tomcat活跃配置的根目录，默认等于CATALINA_HOME。

当CATALINA_HOME和CATALINA_BASE路径分开时，CATALINA_HOME目录只需要含有bin、lib目录，其他目录非必需，CATALINA_BASE目录只需要含有conf、logs、webapps、work、temp目录，其他目录非必需。可以只有1个CATALINA_HOME目录，多个CATALINA_BASE目录，相当于一个tomcat安装目录部署多个tomcat实例。升级和维护Tomcat，只会变更CATALINA_HOME目录或变更CATALINA_HOME指向的目录，其他的Tomcat实例目录不会受影响。

# 配置
1. 创建软链接 /tomcat_home 指向 tomcat二进制分发版本解压目录，作为CATALINA_HOME目录，创建 /tomcat 目录作为CATALINA_BASE目录。
2. 将 CATALINA_HOME 目录的 conf、logs、webapps、work、temp 目录拷贝到 CATALINA_BASE 目录。
3. 修改 /tomcat/conf/server.xml 里面的监听端口配置 和 其他项目相关配置。
4. 新增 /tomcat/bin/setenv.sh 文件，配置运行参数（可选）
5. 增加启动、关闭脚本

   在 /tomcat/bin目录新增 exec.sh 脚本，内容如下：
   
   ```
   #!/bin/bash

   TOMCAT_HOME="$(dirname $0)/.."
   cd $TOMCAT_HOME && TOMCAT_HOME=$PWD && cd - &> /dev/null
   export TOMCAT_HOME

   export CATALINA_HOME="/tomcat_home"
   export CATALINA_BASE="$(readlink -f "$TOMCAT_HOME")"

   echo "JAVA_HOME set to $JAVA_HOME"
   echo "CATALINA_BASE set to $CATALINA_BASE"
   echo "CATALINA_HOME set to $CATALINA_HOME"

   $CATALINA_HOME/bin/"$(basename "$0")" "$@"
   ```

   在bin目录创建软链接startup.sh、shutdown.sh 链接到exec.sh
   
   ``` 
   cd /tomcat/bin
   ln -s exec.sh startup.sh
   ln -s exec.sh shutdown.sh
   ```

可把以上过程写在tomcat_instance_init.sh脚本，方便自动化处理。  
 
# 启动
     /tomcat/bin/startup.sh 

# 其他
更新Tomcat版本时，更新 软链接 /tomcat_home 指向新的Tomcat分发版本。

放在 /tomcat_home/lib 目录的jar，会被全部实例共享。

# 参见
[ Multiple Tomcat Instances | Roberto Cortez Java Blog](http://www.radcortez.com/multiple-tomcat-instances/)

[Tomcat多实例单应用部署方案 - 文章 - 伯乐在线](http://blog.jobbole.com/109347/)

[Tomcat RUNNING.txt](https://tomcat.apache.org/tomcat-8.5-doc/RUNNING.txt)