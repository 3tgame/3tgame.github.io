---
published: true
layout: post
title: 浏览器出现Status为canceled的请求
date: 2017-06-15T16:18:18.000Z
categories:
  - http javascript
---
# 问题
使用微信授权登录后出现没有登录成功的问题，使用微信开发者工具，查看HTTP请求，发现有一个请求被cancel掉，如下所示

![canceled_request.png](/img/canceled_request.png)

查看请求详细信息，看到错误提示“Provisional headers are shown”：

![canceled_request_detail.png](/img/canceled_request_detail.png)
# 分析
一开始怀疑是微信限制window.location.href的赋值操作，因为这个跳转是由 window.location.href = xxxx 引起的，排查一段时间后，发现这个跳转极少概率可以执行成功，就是微信没有限制window.location.href的赋值操作。

搜索错误的信息“Provisional headers are shown”，有说请求可能被浏览器插件（如AdBlock）屏蔽了，但我不开启这个插件还是有这个问题，况且Chrome浏览器的插件不会影响微信开发工具。有说可在Chrome浏览器地址栏输入 chrome://net-internals，即可查看Event详细信息，可是微信虽然使用Chrome内核，但不能使用该指令。

结合Nginx日志排查，发现有时候服务端不能收到请求，有时候服务端能收到请求。服务端收到请求，又分为2种情况：1. 服务端没处理完，客户端就断开请求连接，服务端返回499。2. 服务端处理完，返回301跳转。
如果是客户端拦截的话，这拦截时间有点不确定，为什么不一开始在客户端请求前进行拦截，要让请求发送出去。

后面怀疑是因为js脚本没加载完就执行，可是改成加载完js，再执行 window.location.href，还是一样的问题。

暴力通过 setTimeout("window.location.href = '" + url + "'", 300); 延迟执行跳转，则没问题。缩短延迟执行时间，还是有问题。

在js中执行 window.location.href = url1; window.location.href = url2;  发现最后只有url2的请求。估计是2个操作间隔太短，导致第一个请求没发送就拦截。

后来想到页面加载完后会跳转到 http://www.xxx.com/#/，这也是修改location，导致浏览器认为之前没完成的location跳转没必要，就终止掉那次跳转请求。

# 验证

编写以下页面进行测试：

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>test</title>
    <script type="text/javascript">
        function onClick(){
            window.location.href = "https://www.baidu.com";
            setTimeout("window.location.href = 'https://www.baidu.com/#a'", 10);
            //setTimeout("window.location.hash = 'ab'", 10);
        }
    </script>
</head>
<body>
    <button  type="button" onclick="onClick()" >点我</button>
</body>
</html>
```

先执行  window.location.href = "https://www.baidu.com" 进行跳转，使用setTimeout() 延迟执行 window.location.href = 'https://www.baidu.com/#a'，以达到在跳转到baidu的过程中再进行跳转。从控制台可看到第一个跳转请求被cancel。

通过 setTimeout("window.location.hash = 'ab'", 10) 的执行效果，也可验证 修改location的hash也会导致之前未完成的location跳转被终止。

加大延迟执行间隔，如果300毫秒后再执行第2次跳转，则第1次跳转不会被cancel，因为第1次跳转的请求已被响应返回。

# 解决方法

在跳转没完成过程中，不要进行第二次跳转（设置location或设置location.hash）。在单页的网页应用中，需要设置location.hash来区分不同页面，这就需要在跳转后再设置location.hash，或者设置location.hash后再跳转。
