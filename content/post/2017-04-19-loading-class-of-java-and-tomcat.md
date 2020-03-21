---
published: true
layout: post
title: Java和Tomcat的类加载
date: 2017-04-19T16:18:18.000Z
categories:
  - tomcat java
---
# Java的类加载
类从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期包括：加载、验证、准备、解析、初始化、使用和卸载七个阶段。

## Java的类加载器
层次关系如下：

     启动类加载器
             |
     扩展类加载器
             |
     应用程序类加载器
             |
     自定义类加载器

### 启动类加载器：
Bootstrap ClassLoader，跟上面相同。它负责加载存放在JDK\jre\li(JDK 代表 JDK 的安装目录，下同)下，或被-Xbootclasspath参数指定的路径中的，并且能被虚拟机识别的类库（如 rt.jar，所有的java.*开头的类均被 Bootstrap ClassLoader 加载）。启动类加载器是无法被 Java 程序直接引用的。
### 扩展类加载器：
Extension ClassLoader，该加载器由sun.misc.Launcher$ExtClassLoader实现，它负责加载JDK\jre\lib\ext目录中，或者由 java.ext.dirs 系统变量指定的路径中的所有类库（如javax.*开头的类），开发者可以直接使用扩展类加载器。
### 应用程序类加载器：
Application ClassLoader，该类加载器由 sun.misc.Launcher$AppClassLoader 来实现，它负责加载用户类路径（ClassPath）所指定的类，开发者可以直接使用该类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

## 其他
在 java 启动参数中添加 -verbose:class，即可从日志中看到class 加载相关的日志。

# Tomcat的类加载
## Tomcat的类加载器
在Tomcat启动时创建4种加载器，父子关系如下：

       Bootstrap
          |
       System
          |
       Common
       /     \
     Webapp1   Webapp2 ...

### Bootstrap
该class loader包含Java Virtual Machine提供的基本运行时class和系统扩展目录（$JAVA_HOME/jre/lib/ext）的JAR文件的class。
### System
该class loader加载$CATALINA_HOME/bin/bootstrap.jar、$CATALINA_BASE/bin/tomcat-juli.jar 或 $CATALINA_HOME/bin/tomcat-juli.jar、$CATALINA_HOME/bin/commons-daemon.jar，这些jar都是在$CATALINA_HOME/bin/catalina.sh中指定。
### Common
该class loader包含对Tomcat内部class和所有web 应用都可见的class。该class loader搜索的路径由 $CATALINA_BASE/conf/catalina.properties 文件的 common.loader属性指定。默认设置按以下顺序搜索：

     unpacked classes and resources in $CATALINA_BASE/lib
     JAR files in $CATALINA_BASE/lib
     unpacked classes and resources in $CATALINA_HOME/lib
     JAR files in $CATALINA_HOME/lib
     
### WebappX
为每一个web应用创建的class loader，web应用的 /WEB-INF/classes 目录的所有unpacked classes和resources、/WEB-INF/lib 目录的JAR文件的classes和resources，只对当前web应用可见。

## 类加载顺序
对于一个web应用，class和resource加载不同于默认的Java委派模型，按以下顺序查找：

     Bootstrap classes of your JVM
     /WEB-INF/classes of your web application
     /WEB-INF/lib/*.jar of your web application
     System class loader classes
     Common class loader classes

更多参见 [Apache Tomcat 8 (8.5.14) - Class Loader HOW-TO](https://tomcat.apache.org/tomcat-8.5-doc/class-loader-howto.html)

## 其他
在 /tomcat_home/conf/catalina.properties   可指定类加载相关属性

如果需要某些servlet在web应用启动时加载，可在 conf/web.xml 配置如下：

     <web-app>  
       <servlet>  
            <servlet-name>servlet1</servlet-name>  
            <servlet-class>com.javatpoint.FirstServlet</servlet-class>  
            <load-on-startup>0</load-on-startup> //value given 0(zero)
       </servlet>
     </web-app>

load-on-startup 表明该servlet在启动web 应用时，会被加载、初始化并执行init()方法。该元素的值表示加载的顺序。值越低优先级越高。

## 问题
在不停止Tomcat时，发布更新web应用目录下的class文件后，日志输出错误信息“java.lang.IllegalStateException: Method [myInfo] was discovered in the .class file but cannot be resolved in the class object”。

原因：这是因为JVM是按需加载class，在需要某个class时才动态加载该class，当class文件被更新，类定义不一致，导致加载类失败。如果执行请求导致该class已被加载，接着在不停Tomcat的情况下更新class文件，不会出现该错误。因为该class已加载不会导致加载新的class定义。

解决方法：可以 在停掉Tomcat后，再发布更新class文件。或者更新回旧版本的class，因为之前加载失败，再次请求，会导致再次加载class。

# 参见
[Java Language Specification - Chapter 12. Execution](http://docs.oracle.com/javase/specs/jls/se8/html/jls-12.html#jls-12.2)

[Java Virtual MachineSpecification - Chapter 5. Loading, Linking, and Initializing](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.3)

[类加载机制 - 深入理解 Java 虚拟机 - 极客学院Wiki](http://wiki.jikexueyuan.com/project/java-vm/class-loading-mechanism.html)

[浅议tomcat与classloader - 笨狐狸 - 博客频道 - CSDN.NET](http://blog.csdn.net/liweisnake/article/details/8470285)
