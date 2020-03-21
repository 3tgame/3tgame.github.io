---
layout: post
title:  "使用Markdown和DropBox第3方服务搭建静态博客"
date:   2013-09-01 23:59:18
categories: [blog]
---

# 注册Scriptogr服务

到[Scriptogr.am](http://scriptogr.am/)注册服务。注册完成会在Dropbox目录下创建目录**_应用/scriptogram_**,里面的posts目录用于放置Markdown文件。

# 编写Markdown

在*Dropbox/应用/scriptogram/posts*目录创建Markdown文件**2013-09-01.md**。使用Markdown语法编写文章。可使用Markdown在线编辑工具[markable.in](http://markable.in/editor/) 或[dingus](http://daringfireball.net/projects/markdown/dingus)进行所见即所得的编写，可到[Markdown的网站](http://daringfireball.net/projects/markdown/syntax)查看Markdown的语法。

## 加入Scriptogr服务需要的说明信息

按照启用Scriptogr服务后，自动创建的一篇介绍文章[This is your blog, delivered by scriptogr.am](http://scriptogr.am/3tgame/post/this-is-your-blog-delivered-by-scriptogr.am)的说明，在编写的Markdown文件前加入如下内容:
    
    Date: 2011-12-31 12:31
    Title: This is your blog, delivered by scriptogr.am
  

> 其中Title是必需的，Date不是必需的。

# 同步Markdown文件到Scriptogr

进入[Scriptogr网站的控制面板](http://scriptogr.am/dashboard#dashboard)，点击同步（Synchronize）按钮。在登录[Scriptogr个人博客域名](http://scriptogr.am/3tgame)，即可刚发布的文章。