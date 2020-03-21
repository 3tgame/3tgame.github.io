---
published: true
layout: post
title: 在不同服务器共享Session
date: 2017-04-13T16:18:18.000Z
categories:
  - tomcat session
---
部署负载均衡时，使用一台Nginx作为前端服务器，几台Tomcat作为后端服务器。Nginx根据策略把请求分发给Tomcat服务器。默认Tomcat服务器的session是不可跨服务器共享的，如果同一个用户的不同请求被分发到不同的Tomcat服务器，怎会造成session丢失，要用户重新登录。

# 方法1：upstream ip_hash
ip_hash 是根据请求的IP进行分发，对于同一个IP的请求会转发到同一台Tomcat。

ngxin配置如下：
```
upstream tomcat {
    server 127.0.0.1:8081 ;
    server 127.0.0.1:8082 ;
    ip_hash;
}
server {
    listen       80;
    server_name  localhost;

    location ~ .*\.(do|jsp|action)?$ {
    proxy_redirect          off;
        proxy_set_header        Host            $host;
        proxy_set_header        X-Real-IP       $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        root   html;
        index  index.html index.htm;
        proxy_pass http://tomcat;
    }
}
```

生产环境不推荐使用，原因如下：
1. Nginx不是最前端的服务器，如还是要Squid作为前端缓存，则Nginx实际是拿Squid服务器的IP，达不到根据客户端请求IP分流的效果。
2. 有些机构是使用动态IP或有多个出口IP，用户访问会切换IP，则无法将某个用户的请求固定到同一个Tomcat服务器。

# 方法2：nginx_upstream_jvm_route
[nginx_upstream_jvm_route](https://github.com/nulab/nginx-upstream-jvm-route) 是一个nginx扩展模块，使用cookie实现session黏合。nginx_upstream_jvm_route在用户第一次请求转发到后端server时，会把响应的server标识添加到name为JSESSIONID的cookie中。下一次请求时nginx会根据JSESSIONID决定转发给哪一台后端server。

Nginx配置如下：
```
upstream  tomcats_jvm_route {
    server   127.0.0.1:8081 srun_id=tomcat01;
    server   127.0.0.1:8082 srun_id=tomcat02;
    jvm_route $cookie_JSESSIONID|sessionid reverse;
}
```

Tomcat的server.xml配置如下：
```
<Engine name="Catalina" defaultHost="localhost" jvmRoute="tomcat01">
<Engine name="Catalina" defaultHost="localhost" jvmRoute="tomcat02">
```

配置后，请求的name为JSESSIONID的cookie末尾被添加服务器标识，如下：
     JSESSIONID=F8A9463760E0BC88B7919F4C35801E5B.tomcat02;

生产环境不推荐使用，原因如下：
1. 根据tomcat的特性，当server.xml配置文件中加了jvmRoute值后，会给sessionid加上jvmRoute值的后缀，根据这一特性，nginx_upstream_jvm_route对每次访问请求中的sessionId的值，自动匹配对应的server。这样就会使得每次都访问到同一个tomcat，这样就解决了访问不同tomcat节点，session发生变化的问题。但是这种方式就会有当一直访问的tomcat节点挂掉之后，根据负载的原理，将会访问其它节点，就会造成session发生变化，需重新登录的问题。

# 方法3：使用redis共享session
以上2种方法都是把Session存储在Tomcat容器内，当请求从1个Tomcat转发到另一个Tomcat，Session就会失效，因为Session在各个Tomcat中不共享。如果使用redis等外服缓存系统存储Session，则可以在各个Tomcat实例间共享Session。

Java框架使用 [redisson](https://github.com/redisson/redisson) 来实现Tomcat Session管理。配置方法，可参见 [Tomcat Redis Session Manager](https://github.com/redisson/redisson/wiki/14.-Integration-with-frameworks#145-tomcat-redis-session-manager) 的说明。

## 配置
1. 编辑 TOMCAT_BASE/conf/context.xml 文件，添加以下内容： 
   ```
   <Manager className="org.redisson.tomcat.RedissonSessionManager" configPath="${catalina.base}/redisson-config.json" />
   ```
2. 把 redisson-all-2.8.1.jar 和 redisson-tomcat-7-2.8.1.jar 放到 TOMCAT_BASE/lib 目录。
3. 按照 [Single instance mode](https://github.com/redisson/redisson/wiki/2.-Configuration#262-single-instance-json-yaml-and-spring-xml-config-format) 的说明，编辑 json格式的 redisson-config.json 文件。

# 参见
[Nginx+tomcat+redis集群之session跟踪（nginx_upstream_jvm_route） - 建均笔记](http://www.wujianjun.org/2016/03/18/nginx-upstream-jvm-route/)

[Nginx+tomcat+redis集群之session共享 - 建均笔记](http://www.wujianjun.org/2016/03/17/tomcat-redis-session/)
