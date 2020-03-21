---
published: true
layout: post
title: IE9跨域发送POST请求，服务端获取不到POST参数
date: 2017-03-10T15:30:18.000Z
categories:
  - 跨域
---
## 服务端写日志打印POST参数

如使用PHP框架，则可使用 `error_log(print_r($_POST, true));`  打印请求POST参数。
使用 `error_log(file_get_contents("php://input"));`  打印请求body。

## 使用IE9发送跨域POST请求

打开IE（如果使用Win10，可以通过“Windwos附件 --> Internet Explorer” ），如果当前版本不是IE9，按F12，打开IE的开发者模式。请求带有参数的跨域POST请求的访问连接。

可看到输出数组为空，body则为username=xxxx&password=yyyy&captcha=etnmb。

如果业务逻辑有使用POST参数，则提示没有传递username和password参数。

## 使用IE10、或者Chrome发送跨域POST请求

发现输出为

    Array
	(
		[account] => xxxx
		[password] => yyy
		[captcha] => etnmb
	)

业务逻辑没有报错。

## 对比使用IE9，IE10的请求

发现IE9的请求没有发送Content-Type Header。

## 实验是否是因为没有发送Content-Type Header导致

使用Postman模拟请求，Body选择“x-www-form-urlencoded”，Header取消勾选“Content-Type”，点击“Code”，即可看到发送内容如下：

	POST /store/server/public/admin/auth/login HTTP/1.1
	Host: 192.168.31.99
	Cache-Control: no-cache
	Postman-Token: 0f0d1b4c-e1e8-9e20-3fed-4663b6444ca4

	account=xxxx&password=yyyy&captcha=etnmb

点击“发送”，看到服务端输出日志中$_POST数组为空。

勾选“Content-type”，点击“Code”，即可看到发送内容如下：

	POST /store/server/public/admin/auth/login HTTP/1.1
	Host: 192.168.31.99
	Content-Type: application/x-www-form-urlencoded
	Cache-Control: no-cache
	Postman-Token: 0f0d1b4c-e1e8-9e20-3fed-4663b6444ca4

	account=xxxx&password=yyyy&captcha=etnmb

点击“发送”，看到服务端输出日志中$_POST数组不为空。

## 分析

实验发现JAVA框架也有这样的问题。

可以确定就是没有发送“Content-Type” Header导致PHP框架不能正确设置$_POST参数。

打算在前端框架设置请求“Content-Type”Header，试验发现在IE9中不能设置。说明可参见[XDomainRequest – Restrictions, Limitations and Workarounds](https://blogs.msdn.microsoft.com/ieinternals/2010/05/13/xdomainrequest-restrictions-limitations-and-workarounds/) 中的“Unfortunately, when we fixed this problem in a later IE8 Beta, we went a bit too far; we restricted the content type to text/plain but didn’t allow the caller to specify that the data was in application/x-www-urlencoded form. This is problematic because server-side frameworks (e.g. ASP, ASPNET, etc) will only automatically parse a request’s fields into name-value pairs if the x-www-urlencoded content type is specified.”

[XDomainRequest – Restrictions, Limitations and Workarounds](https://blogs.msdn.microsoft.com/ieinternals/2010/05/13/xdomainrequest-restrictions-limitations-and-workarounds/) 中的“As of 2014, XDomainRequest doesn’t appear to send any Content-Type header at all. It’s not clear to me when this changed.”也确认了IE9存在发送跨域请求不会发送“Content-Type”Header这样的问题。

[PHP:$_POST -Manual](http://php.net/manual/zh/reserved.variables.post.php) 中也说明$_POST是“当 HTTP POST 请求的 Content-Type 是 **application/x-www-form-urlencoded** 或 **multipart/form-data** 时，通过 HTTP POST 方法传递给当前脚本中变量的关联数组。” ，所有没有“Content-Type”Header时，$_POST数组为空。

打算在业务框架使用自定义Header“X-Content-Type”进行处理，可是实验发现IE9不能设置自定义Header，可参见[XDomainRequest – Restrictions, Limitations and Workarounds](https://blogs.msdn.microsoft.com/ieinternals/2010/05/13/xdomainrequest-restrictions-limitations-and-workarounds/) 的“No custom headers may be added to the request”。

## 解决方法

[XDomainRequest – Restrictions, Limitations and Workarounds](https://blogs.msdn.microsoft.com/ieinternals/2010/05/13/xdomainrequest-restrictions-limitations-and-workarounds/) 也说明了以下解决方法：

“To workaround this issue, server code that currently processes HTML Forms must be rewritten to manually parse the request body into name-value pairs when receiving requests from XDomainRequest objects. This makes adding support for the XDomainRequest object more difficult than it would be otherwise.”

对于PHP框架，参见[How to get POST parameters with wrong header](http://stackoverflow.com/questions/8183397/how-to-get-post-parameters-with-wrong-header)，可使用以下方法解决:

`if (count($_POST) == 0)
{
    parse_str(file_get_contents("php://input"), $_POST);
}
`

也可根据User Agent判断是否为IE9（"Mozilla/5.0 (compatible; **MSIE 9.0**; Windows NT 6.1; Trident/5.0)"），进行特殊处理。

`$user_agent = $request->header('User-Agent');
if(preg_match('/MSIE\s9/i',$user_agent))
{
    parse_str(file_get_contents("php://input"), $_POST);
}
`

## 参见

[XDomainRequest – Restrictions, Limitations and Workarounds](https://blogs.msdn.microsoft.com/ieinternals/2010/05/13/xdomainrequest-restrictions-limitations-and-workarounds/)

[PHP:$_POST -Manual](http://php.net/manual/zh/reserved.variables.post.php)

[How to get POST parameters with wrong header](http://stackoverflow.com/questions/8183397/how-to-get-post-parameters-with-wrong-header)
